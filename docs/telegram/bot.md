---
Author: pony@diynova.com
Date: 2022-03-21 10:36:59
LastEditors: pony@diynova.com
LastEditTime: 2022-03-21 10:51:44
FilePath: /notes/docs/telegram/bot.md
Description: 添加验证机器人
---

# 添加验证机器人

- 添加机器人为好友 @join_captcha_bot
- 将机器人 join_captcha_bot 加入群组， 并设置管理员权限，可以封禁用户，和删除消息
- 开启验证功能,聊天界面输入 /enable
- 如需要设置保护时间： /time
- 设置语言 /language zh_cn ...
- 设置验证难度 /difficulty 5
- 设置验证码格式 /captcha_mode ..


```
Hello, I am a Bot that sends an image captcha for each new user that joins a group, and kicks anyone that can't solve the captcha within a specified time.

If a user tries to join the group 5 times in a row and never solves the captcha, I will assume that this "user" is a bot, and it will be banned. Also, any message that contains an URL sent by a new "user" before the captcha is completed will be considered spam and will be deleted.

Remember to give me administration privileges to kick-ban users and remove messages.

Check /help command for more information about my usage.

Am I useful? Check /about command and consider making a donation to keep me active.

你好，我是一个Bot，它为每个加入群组的新用户发送一个图像验证码，并在指定时间内踢出任何无法解决验证码的人。

如果一个用户连续 5 次尝试加入该组并且从未解决验证码，我会假设这个“用户”是一个机器人，它将被禁止。 此外，任何包含新“用户”在验证码完成之前发送的 URL 的邮件都将被视为垃圾邮件并被删除。

请记住授予我管理权限以禁止用户和删除消息。

检查 /help 命令以获取有关我的用法的更多信息。

我有用吗？ 检查 /about 命令并考虑捐款以使我保持活跃。
```


```
Bot help:
————————————————
- I am a Bot that sends a captcha for each new user that joins a group, and kick any of them that can't solve the captcha within a specified time.

- If a user tries to join the group 5 times in a row and never solves the captcha, I will assume that the "user" is a bot, and it will be banned.

- Any message that contains an URL that has been sent by a new "user" before captcha is completed will be considered spam and will be deleted.

- You need to grant me Administration rights so I can kick users and remove messages.

- To preserve a clean group, I auto-remove all messages related to me when a captcha is not solved and the user was kicked.

- The time that new users have to solve the captcha is 5 minutes by default, but it can be configured using the command /time.

- You can turn captcha protection on/off using the commands /enable and /disable.

- Configuration commands can only be used by group Administrators.

- You can change the language that I speak, using the command /language.

- You can configure captcha difficulty level using command /difficulty.

- You can set captcha to use full numbers and letters A–Z, or numbers and letters A–F, or just numbers (default), or a math equation to be solved, or a custom poll, or a button to be pressed, using command /captcha_mode.

- You can configure a custom welcome message with command /welcome_msg.

- You can enable an option to let me apply restriction to new joined users to send non-text messages using command /restrict_non_text.

- If the Bot is Private, allow groups with command /allowgroup.

- You can configure a group from private Bot chat through /connect command.

- You can block users to send any message that contains an URL/link in a group by /url_disable command.

- Check /commands to get a list of all avaliable commands, and a short description of all of them.


机器人帮助：
—————————————————
- 我是一个机器人，它为每个加入群组的新用户发送验证码，并在指定时间内踢出任何无法解决验证码的用户。

- 如果一个用户连续 5 次尝试加入群组并且从未解决验证码，我将假设“用户”是一个机器人，它将被禁止。

- 任何包含在验证码完成之前由新“用户”发送的 URL 的邮件都将被视为垃圾邮件并将被删除。

- 您需要授予我管理权限，这样我才能踢出用户并删除消息。

- 为了保持一个干净的组，当验证码未解决并且用户被踢出时，我会自动删除与我相关的所有消息。

- 新用户解决验证码的时间默认为 5 分钟，但可以使用命令 /time 进行配置。

- 您可以使用 /enable 和 /disable 命令打开/关闭验证码保护。

- 配置命令只能由组管理员使用。

- 您可以使用命令 /language 更改我说的语言。

- 您可以使用命令 /difficulty 配置验证码难度级别。

- 您可以将验证码设置为使用完整的数字和字母 A-Z，或数字和字母 A-F，或仅使用数字（默认），或要解决的数学方程，或自定义投票，或要按下的按钮，使用命令 /captcha_mode。

- 您可以使用命令 /welcome_msg 配置自定义欢迎消息。

- 您可以启用一个选项，让我对新加入的用户应用限制以使用命令 /restrict_non_text 发送非文本消息。

- 如果 Bot 是私有的，使用命令 /allowgroup 允许组。

- 您可以通过 /connect 命令从私人 Bot 聊天中配置群组。

- 您可以通过 /url_disable 命令阻止用户发送包含组中 URL/链接的任何消息。

- 检查 /commands 以获取所有可用命令的列表，以及所有命令的简短描述。
```