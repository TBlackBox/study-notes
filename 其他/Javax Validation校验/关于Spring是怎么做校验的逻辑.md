# 简介
下面简单介绍spring是怎么做检验逻辑。


## 过程
1. Spring会为Controller添加一个拦截器, 拦截器叫 : MethodValidationInterceptor
2. 当请求数据时, 会经过拦截器, 拦截器判断 Çlass 或者 方法上是否标记有 Validated 注解
3. 当请求被AOP拦截并执行校验逻辑, 并存在校验不通过情况时, 则会抛出 ConstraintViolationException 异常


源码：
```java
/**
 * An AOP Alliance {@link MethodInterceptor} implementation that delegates to a
 * JSR-303 provider for performing method-level validation on annotated methods.
 *
 * <p>Applicable methods have JSR-303 constraint annotations on their parameters
 * and/or on their return value (in the latter case specified at the method level,
 * typically as inline annotation).
 *
 * <p>E.g.: {@code public @NotNull Object myValidMethod(@NotNull String arg1, @Max(10) int arg2)}
 *
 * <p>Validation groups can be specified through Spring's {@link Validated} annotation
 * at the type level of the containing target class, applying to all public service methods
 * of that class. By default, JSR-303 will validate against its default group only.
 *
 * <p>As of Spring 5.0, this functionality requires a Bean Validation 1.1 provider.
 *
 * @author Juergen Hoeller
 * @since 3.1
 * @see MethodValidationPostProcessor
 * @see javax.validation.executable.ExecutableValidator
 */
public class MethodValidationInterceptor implements MethodInterceptor {

    private final Validator validator;


    /**
     * Create a new MethodValidationInterceptor using a default JSR-303 validator underneath.
     */
    public MethodValidationInterceptor() {
        this(Validation.buildDefaultValidatorFactory());
    }

    /**
     * Create a new MethodValidationInterceptor using the given JSR-303 ValidatorFactory.
     * @param validatorFactory the JSR-303 ValidatorFactory to use
     */
    public MethodValidationInterceptor(ValidatorFactory validatorFactory) {
        this(validatorFactory.getValidator());
    }

    /**
     * Create a new MethodValidationInterceptor using the given JSR-303 Validator.
     * @param validator the JSR-303 Validator to use
     */
    public MethodValidationInterceptor(Validator validator) {
        this.validator = validator;
    }


    @Override
    @SuppressWarnings("unchecked")
    public Object invoke(MethodInvocation invocation) throws Throwable {
        // Avoid Validator invocation on FactoryBean.getObjectType/isSingleton
        if (isFactoryBeanMetadataMethod(invocation.getMethod())) {
            return invocation.proceed();
        }

        Class<?>[] groups = determineValidationGroups(invocation);

        // Standard Bean Validation 1.1 API
        ExecutableValidator execVal = this.validator.forExecutables();
        Method methodToValidate = invocation.getMethod();
        Set<ConstraintViolation<Object>> result;

        try {
            result = execVal.validateParameters(
                    invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
        }
        catch (IllegalArgumentException ex) {
            // Probably a generic type mismatch between interface and impl as reported in SPR-12237 / HV-1011
            // Let's try to find the bridged method on the implementation class...
            methodToValidate = BridgeMethodResolver.findBridgedMethod(
                    ClassUtils.getMostSpecificMethod(invocation.getMethod(), invocation.getThis().getClass()));
            result = execVal.validateParameters(
                    invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
        }
        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }

        Object returnValue = invocation.proceed();

        result = execVal.validateReturnValue(invocation.getThis(), methodToValidate, returnValue, groups);
        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }

        return returnValue;
    }

    private boolean isFactoryBeanMetadataMethod(Method method) {
        Class<?> clazz = method.getDeclaringClass();

        // Call from interface-based proxy handle, allowing for an efficient check?
        if (clazz.isInterface()) {
            return ((clazz == FactoryBean.class || clazz == SmartFactoryBean.class) &&
                    !method.getName().equals("getObject"));
        }

        // Call from CGLIB proxy handle, potentially implementing a FactoryBean method?
        Class<?> factoryBeanType = null;
        if (SmartFactoryBean.class.isAssignableFrom(clazz)) {
            factoryBeanType = SmartFactoryBean.class;
        }
        else if (FactoryBean.class.isAssignableFrom(clazz)) {
            factoryBeanType = FactoryBean.class;
        }
        return (factoryBeanType != null && !method.getName().equals("getObject") &&
                ClassUtils.hasMethod(factoryBeanType, method.getName(), method.getParameterTypes()));
    }

    /**
     * Determine the validation groups to validate against for the given method invocation.
     * <p>Default are the validation groups as specified in the {@link Validated} annotation
     * on the containing target class of the method.
     * @param invocation the current MethodInvocation
     * @return the applicable validation groups as a Class array
     */
    protected Class<?>[] determineValidationGroups(MethodInvocation invocation) {
        Validated validatedAnn = AnnotationUtils.findAnnotation(invocation.getMethod(), Validated.class);
        if (validatedAnn == null) {
            validatedAnn = AnnotationUtils.findAnnotation(invocation.getThis().getClass(), Validated.class);
        }
        return (validatedAnn != null ? validatedAnn.value() : new Class<?>[0]);
    }

}
```