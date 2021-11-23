# 简介
在Spring中，filter默认继承OncePerRequestFilter,为什么诶？

## 部分源码
```java
@Override
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		if (!(request instanceof HttpServletRequest) || !(response instanceof HttpServletResponse)) {
			throw new ServletException("OncePerRequestFilter just supports HTTP requests");
		}
		HttpServletRequest httpRequest = (HttpServletRequest) request;
		HttpServletResponse httpResponse = (HttpServletResponse) response;

		String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
		boolean hasAlreadyFilteredAttribute = request.getAttribute(alreadyFilteredAttributeName) != null;

		if (skipDispatch(httpRequest) || shouldNotFilter(httpRequest)) {

			// Proceed without invoking this filter...
			filterChain.doFilter(request, response);
		}
		else if (hasAlreadyFilteredAttribute) {

			if (DispatcherType.ERROR.equals(request.getDispatcherType())) {
				doFilterNestedErrorDispatch(httpRequest, httpResponse, filterChain);
				return;
			}

			// Proceed without invoking this filter...
			filterChain.doFilter(request, response);
		}
		else {
			// Do invoke this filter...
			request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);
			try {
				doFilterInternal(httpRequest, httpResponse, filterChain);
			}
			finally {
				// Remove the "already filtered" request attribute for this request.
				request.removeAttribute(alreadyFilteredAttributeName);
			}
		}
}
```

# 原因
OncePerRequestFilter顾名思义，他能够确保在**一次请求只通过一次filter**，而不需要重复执行。大家常识上都认为，一次请求本来就只过一次，为什么还要由此特别限定呢，此方式是为了兼容不同的web container，特意而为之（jsr168），也就是说并不是所有的container都像我们期望的只过滤一次，servlet版本不同，表现也不同：。
如，servlet2.3与servlet2.4也有一定差异
写道

```
在servlet-2.3中，Filter会过滤一切请求，包括服务器内部使用forward转发请求和<%@ include file="/index.jsp"%>的情况。

到了servlet-2.4中Filter默认下只拦截外部提交的请求，forward和include这些内部转发都不会被过滤，但是有时候我们需要 forward的时候也用到Filter。
```
因此，为了兼容各种不同的运行环境和版本，默认filter继承OncePerRequestFilter是一个比较稳妥的选择。
