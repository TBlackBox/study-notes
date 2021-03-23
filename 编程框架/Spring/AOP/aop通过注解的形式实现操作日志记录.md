

## 创建日志的注解
```java
@Retention(RUNTIME)
@Target(METHOD)
public @interface OperationLog {
	
	String name ();
}

```
## 创建切面

```java

@Aspect
@Component
public class OperationLogAspect {

	private static final Logger logger = LoggerFactory.getLogger(OperationLogAspect.class);
	
	@Autowired
	ThreadPoolTaskExecutor threadPoolTaskExecutor;
	
	@Autowired
	OperationLogService operationLogService;
	
	@Pointcut(value = "@annotation(com.cicada.annotation.OperationLog)")
	public void OperatonLog() {
	}
	
	@Before("OperatonLog()")
	public void beforeHandler(JoinPoint joinPoint) throws IOException, ServletException {
		Object target = joinPoint.getTarget();
		String methodName = joinPoint.getSignature().getName();
		Object[] args = joinPoint.getArgs();
		Class<?>[] parameterTypes = ((MethodSignature) joinPoint.getSignature()).getMethod().getParameterTypes();
		logger.debug("操作日志注解拦截到的信息：",target,methodName,args,parameterTypes);
		Method method = null;
		try {
			method = target.getClass().getMethod(methodName, parameterTypes);
			if (method.isBridge()) {
				for (int i = 0; i < args.length; i++) {
					Class<?> genClazz = CommonUtils.getSuperClassGenricType(target.getClass(), 0);
					if (args[i].getClass().isAssignableFrom(genClazz)) {
						parameterTypes[i] = genClazz;
					}
				}
				method = target.getClass().getMethod(methodName, parameterTypes);
			}
		} catch (SecurityException | NoSuchMethodException e) {
			e.printStackTrace();
		}

		OperationLog operationLog = method.getAnnotation(OperationLog.class);
		if (operationLog == null) {
			return; // 未标识 @OperationLog
		}
		
		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		HttpSession session = request.getSession(Boolean.FALSE);
		Object attribute = null;
		if (session == null || (attribute = session.getAttribute(Config.ADMIN_LOGIN)) == null) {
			 return; // 未登录
		}

		String httpMethod = request.getMethod();
		String contentType = request.getHeader(HttpHeaders.CONTENT_TYPE);
		
		// 请求参数 application/x-www-form-urlencoded
		JSONObject params = new JSONObject();
		params.put("params", request.getParameterMap());

		// 请求体 application/json
		String requestBody = null;
		
		if (httpMethod.equalsIgnoreCase("POST") || httpMethod.equalsIgnoreCase("PUT")) {
			if (!StringUtils.isEmpty(contentType)) {
				MediaType mediaType = MediaType.parseMediaType(contentType); // 文件上传
				if (MediaType.MULTIPART_FORM_DATA.equalsTypeAndSubtype(mediaType)) {
					List<JSONObject> files = new LinkedList<>();
					for (Part part : request.getParts()) {
						JSONObject file = new JSONObject();
						file.put("name", part.getSubmittedFileName());
						file.put("size", part.getSize());
						file.put("contentType", part.getContentType());
						
						JSONObject headers = new JSONObject();
						for(String header : part.getHeaderNames()) {
							headers.put(header, part.getHeader(header));
						}
						file.put("headers", headers);
						
						files.add(file);
					}
					params.put("files", files);
				} else if (MediaType.APPLICATION_JSON.equalsTypeAndSubtype(mediaType)) {
					// 必须使用可重用流
					// requestBody = CommonUtils.requestBody(request);
				} else if (MediaType.APPLICATION_FORM_URLENCODED.equalsTypeAndSubtype(mediaType)) {
					// request.getParameterMap()
				}
			}
		}

		// 请求头
		JSONObject headers = new JSONObject();
		Enumeration<String> enumeration = request.getHeaderNames();
		while (enumeration.hasMoreElements()) {
			String name = enumeration.nextElement();
			headers.put(name, request.getHeader(name));
		}

		OperationLogEntity operationLogEntity = new OperationLogEntity();
		operationLogEntity.setAdminId(((Admin) attribute).getId());
		operationLogEntity.setOperation(operationLog.name());
		operationLogEntity.setIp("127.0.0.1");
		operationLogEntity.setParams(params.toJSONString());
		operationLogEntity.setBody(requestBody);
		operationLogEntity.setHeaders(headers.toJSONString());
		operationLogEntity.setUrl(request.getRequestURL().toString());
		operationLogEntity.setContentType(contentType);
		operationLogEntity.setUserAgent(request.getHeader(HttpHeaders.USER_AGENT));
		operationLogEntity.setMethod(httpMethod);
		operationLogEntity.setCreatedDate(LocalDateTime.now());

		this.threadPoolTaskExecutor.execute(() -> {
			if (logger.isDebugEnabled()) {
				logger.debug("记录操作日志:{}", operationLogEntity.toString());
			}
			this.operationLogService.create(operationLogEntity);
		});
	}
}

```

## 使用
```
@OperationLog(name = "获取客服列表")
public Object list(}{

}
```