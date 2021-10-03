# springboot事件

## 声明一个事件
```java
package com.video.api.spring.event;

import org.springframework.context.ApplicationEvent;

import com.video.common.domain.entity.OrderEntity;

public class OrderStealEvent extends ApplicationEvent{
	
	private static final long serialVersionUID = 539195833629482333L;
	private OrderEntity orderEntity;

	public OrderStealEvent(OrderEntity orderEntity) {
		super(orderEntity);
		this.orderEntity = orderEntity;
	}

	public OrderEntity getOrderEntity() {
		return orderEntity;
	}

	public void setOrderEntity(OrderEntity orderEntity) {
		this.orderEntity = orderEntity;
	}
	
}


```

## 监听事件
```java
@Component
public class StealEventListener {
	
	static final Logger LOGGER = LoggerFactory.getLogger(StealEventListener.class);
	
	
	/**
	 * 订单扣量
	 * @param event
	 */
	@EventListener(OrderStealEvent.class)
	public void stealOrder(OrderStealEvent event) {
		OrderEntity orderEntity = event.getOrderEntity();
		LOGGER.info("orderEntyity：{}",orderEntity);
	}
}

```

## 发送一个事件
```java
SpringHelper.applicationContext.publishEvent(new OrderStealEvent(orderEntity));
```

