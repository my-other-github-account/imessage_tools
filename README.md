# iMessage-Tools

imessage-tools is a Python library for accessing and manipulating iMessage data on macOS. This package provides an easy-to-use interface for reading all messages (including hidden ones) from a user's chat.db file, as well as sending iMessages.

Notably, this parses *all* iMessage data, even data that isn't available if you parse chat.db with SQL. You need this on MacOS Ventura and later, because it hides some of the message text there for some reason.

The issue with using SQL to retrieve all iMessage data is that for certain types of messages, the "Text" content is hidden within the "attributedBody" field. This project specifically handles this issue by parsing the "attributedBody" field in order to retrieve the "Text" content, ensuring that all iMessages are retrieved, including those not visible through SQL. This allows the user to have access to a complete and accurate representation of their iMessage history.

# Useful features:

 - Read and send iMessages with a single function call
 - Reads messages not visible through normal SQL queries by parsing attributeBody Blob field (this seems to be necessary to see all messages on MacOS Ventura and later)
 - Easily send messages to Group Chats (read_messages exposes Group Chat IDs, and send_message can accept these as recipients)
 - Really barebones codebase so you can more easily fix things if they inevitably go wrong

# Installation

To install imessage-tools, run the following command:

```
pip install git+https://github.com/my-other-github/imessage_tools.git
```
# Reading Messages

To read messages, you can use the read_messages function. This function takes three required arguments:

- db_location - The path to the chat.db file
- n - The number of messages to return (defaults to 10 if not provided)
- self_number - A string representing your phone number or a label for "you" in the messages. Can be arbitrary.
- human_readable_date - A Boolean indicating whether to return the message timestamps as timestamps or human-readable dates (defaults to human-readable)

To send messages, you can use send_message - which takes:

- message - A string, the message you want to send (will be automatically 'escaped')
- phone_number - the phone number, contact name, or group chat ID (e.g. chat376184224...) you want to send to
- group_chat - a boolean determining whether you are attempting to send a group chat

##### Note 1: The Messages app must be open

##### Note 2: If you are sending to a Group Chat, you need to rename it on the Mac you are using - you can 'rename' it to the same name it already has, but for some reason you can't send to group chats unless they were either created or renamed on your actual Mac (and not your iPhone)

### Output:

The output format of the messages returned by the get_messages function is a list of dictionaries, where each dictionary represents a single message. The dictionary has the following keys:

- "rowid": the unique identifier of the message in the chat.db database
- "body": the text of the message, as it appears in the attributedBody field in the database. If the message is not visible via SQL, it will be decoded and parsed from the attributedBody field using the plistlib library.
- "phone_number": the phone number or id of the person who sent the message, as it appears in the handle.id field in the database
- "is_from_me": a Boolean value indicating whether the message was sent by the user or received from someone else
- "cache_roomname": None if it is not a group chat, otherwise the group ID of the group the message was sent in. Some methods need this guid to send group texts, but for this repository you just use the group name
- "date": the date the message was sent, either as a time stamp or in a human-readable format, depending on the value of the human_readable_date argument passed to the function.

The dictionaries in the list are sorted by date in ascending order, so that the most recent messages are at the end of the list. 

### Example:
```
from imessage_tools import read_messages, print_messages, send_message

# Path to the chat.db file
chat_db = "/Users/<YOUR_HOME>/Library/Messages/chat.db"
# Phone number or label for "you"
self_number = "Me"
# Number of messages to return
n = 10

# Read the messages
messages = read_messages(chat_db, n=n, self_number=self_number, human_readable_date=True)

# Print the messages
print_messages(messages)

# Reply to the last message, regardless of whether an individual, or a group chat messaged you:
if messages[-1]["group_chat_name"]:
    send_message("Hello iMessage!", messages[-1]["group_chat_name"], True)
else:
    send_message("Hello iMessage!", messages[-1]["phone_number"], False)

```

### Closing notes:

Note: If you are sending to a Group Chat, you need to rename it on the Mac you are using - you can 'rename' it to the same name it already has, but for some reason you can't send to group chats unless they were either created or renamed on your actual Mac

Note: This codebase uses some pretty crazy binary parsing that is probably not super stable. This was tested on MacOS Ventura, who knows how it will behave on other systems.