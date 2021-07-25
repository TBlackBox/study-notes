1. 创建tg 机器人

   添加**BotFather**账号，并向他发消息

   怎么添加这个账号

   找个好友：发**@BotFather**即可

2. 下面的命令

```
/start

/newbot

YourNameBot (给你的机器人取名字，以 Bot 结尾，不区分大小写，直接发送过去即可，如果重名了会提示重新输入)
```



3. 会话需要的字符串

   ```
   token (创建机器人时已获得)
   chat_id (聊天ID)
   message (要发送的消息，这个由你输入即可，或者是网站后台程序生成的报表数据等等，支持emoji表情哦)
   ```

4. 获取私发id

   添加**userinfobot**获取你的ID，和上面一样，在任意聊天窗口发送“**@userinfobot**”然后点击这条消息即可打开与**userinfobot**的聊天，发送任意消息给**userinfobot**它会返回你的信息，其中包含一个ID，这就是我们需要的**chat_id**



5. 获取群的chat_id

   上面我们通过**userinfobot**这个机器人获取个人的**chat_id**, 使得机器人可以发消息给个人，群组则没办法通过这种方式获取其ID。

   通过下面的步骤可以获得群组ID:

   1. 在群里发送任意消息
   2. 打开这个网址
   ```
   https://api.telegram.org/bot<替换成你机器人的**token**, 包括尖括号>/getUpdates
   ```
   页面会输出一段JSON
   3. 查找 id: **-xxx** 的一段值，这里的 **-xxx** 就是群组ID，机器人下发消息的时候的chat_id字段使用这个即可发送消息到群组了



测试地址

```
curl -X POST “https://api.telegram.org/bot<token>/sendMessage" -d "chat_id=-xxx&text=不用上班的程序员"
```





# 方法

## kickChatMember

使用此方法将用户踢出群组、超级群组或频道。在超级组和频道的情况下，用户将无法使用邀请链接等自行返回聊天，除非先取消禁止。机器人必须是聊天中的管理员才能工作，并且必须具有适当的管理员权限。成功时返回 True



## unbanChatMember

使用此方法在超级组或频道中取消对先前被踢出的用户的封禁。用户不会自动返回群组或频道，但可以通过链接等方式加入。机器人必须是管理员才能执行此操作。默认情况下，此方法保证在通话后用户不是聊天的成员，但可以加入聊天。因此，如果用户是聊天的成员，他们也将从聊天中删除。如果您不想这样做，请使用参数 only_if_banned。成功时返回 True。





## restrictChatMember

使用此方法限制超级组中的用户。机器人必须是超级组中的管理员才能工作，并且必须具有适当的管理员权限。为所有权限传递 True 以解除用户的限制。成功时返回 True。



## promoteChatMember

使用此方法在超级组或频道中提升或降级用户。机器人必须是聊天中的管理员才能工作，并且必须具有适当的管理员权限。为所有布尔参数传递 False 以降级用户。成功时返回 True。



## setChatAdministratorCustomTitle

使用此方法为机器人提升的超级组中的管理员设置自定义标题。成功返回真



## setChatPermissions

使用此方法为所有成员设置默认聊天权限。机器人必须是组或超级组中的管理员才能工作，并且必须具有 can_restrict_members 管理员权限。成功时返回 True。



## pinChatMessage

使用此方法将消息添加到聊天中的固定消息列表。如果聊天不是私人聊天，机器人必须是聊天中的管理员才能工作，并且必须在超级组中具有“can_pin_messages”管理员权限或在频道中具有“can_edit_messages”管理员权限。成功时返回 True。



## unpinChatMessage

使用此方法从聊天中的固定消息列表中删除消息。如果聊天不是私人聊天，机器人必须是聊天中的管理员才能工作，并且必须在超级组中具有“can_pin_messages”管理员权限或在频道中具有“can_edit_messages”管理员权限。成功时返回 True



## unpinAllChatMessages

使用此方法清除聊天中的固定消息列表。如果聊天不是私人聊天，机器人必须是聊天中的管理员才能工作，并且必须在超级组中具有“can_pin_messages”管理员权限或在频道中具有“can_edit_messages”管理员权限。成功时返回 True。



## leaveChat

使用此方法让您的机器人离开群组、超级群组或频道。成功时返回 True。



## getChat

使用此方法可获取有关聊天的最新信息（一对一对话的用户当前名称、用户当前用户名、群组或频道等）。成功时返回一个 Chat 对象。



## getChatAdministrators

使用此方法获取聊天中的管理员列表。成功时，返回一个 ChatMember 对象数组，其中包含有关除其他机器人之外的所有聊天管理员的信息。如果聊天是群组或超级群组且未指定管理员，则只会返回创建者





## getChatMembersCount

使用此方法获取聊天中的成员数量。成功返回 Int



## getChatMember

使用此方法获取有关聊天成员的信息。成功时返回 ChatMember 对象。



## setWebhook

使用此方法指定 url 并通过传出 webhook 接收传入更新。每当机器人有更新时，我们都会向指定的 url 发送 HTTPS POST 请求，其中包含一个 JSON 序列化的更新。如果请求不成功，我们将在合理尝试后放弃。成功时返回 True。

如果您想确保 Webhook 请求来自 Telegram，我们建议在 URL 中使用秘密路径，例如https://www.example.com/<令牌>。由于没有其他人知道您的机器人的令牌，因此您可以非常确定是我们。



## deleteWebhook

如果您决定切换回 getUpdates，请使用此方法删除 webhook 集成。成功时返回 True

## WebhookInfo

包含有关 Webhook 当前状态的信息
