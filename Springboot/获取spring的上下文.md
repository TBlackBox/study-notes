
通过实现`ApplicationContextAware`获取spring容器的上下文

```
package com.video.common.spring;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class SpringHelper implements ApplicationContextAware {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(SpringHelper.class);
	
	public static ApplicationContext applicationContext;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		if (LOGGER.isDebugEnabled()) {
			LOGGER.debug("applicationContext = {}", applicationContext);
		}
		SpringHelper.applicationContext = applicationContext;
	}
}

```