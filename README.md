# main.py-
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext
import requests

# Initial values
OWNER_ID = 7691641018  # Bot owner's ID
GROUP_IDS = []  # Initially empty, groups are added through a command

allowed_users = {}
allow_all_enabled = False  # Temporary permission flag for all members
saved_message = ""  # Колдонуучунун сакталган билдирүүсү

def set_group(update: Update, context: CallbackContext):
    global GROUP_IDS
    if update.effective_user.id != OWNER_ID:
        return
    
    try:
        group_id = int(context.args[0])
    except (IndexError, ValueError):
        update.message.reply_text("Usage: /setgroup {group ID}")
        return
    
    if group_id not in GROUP_IDS:
        GROUP_IDS.append(group_id)
        update.message.reply_text(f"Group ID successfully added: {group_id}")
    else:
        update.message.reply_text(f"Group ID {group_id} is already added.")

def remove_group(update: Update, context: CallbackContext):
    global GROUP_IDS
    if update.effective_user.id != OWNER_ID:
        return
    
    try:
        group_id = int(context.args[0])
    except (IndexError, ValueError):
        update.message.reply_text("Usage: /removegroup {group ID}")
        return
    
    if group_id in GROUP_IDS:
        GROUP_IDS.remove(group_id)
        update.message.reply_text(f"Group ID {group_id} successfully removed.")
    else:
        update.message.reply_text(f"Group ID {group_id} not found.")

def allow(update: Update, context: CallbackContext):
    if update.effective_user.id != OWNER_ID:
        return
    
    try:
        user_id = int(context.args[0])
        amount = int(context.args[1])
    except (IndexError, ValueError):
        update.message.reply_text("Use the command in the following format: /allow {user ID} {number of requests}")
        return
    
    allowed_users[user_id] = amount
    update.message.reply_text(f"User {user_id} has been allowed {amount} requests.")

def allow_all(update: Update, context: CallbackContext):
    global allow_all_enabled
    if update.effective_user.id != OWNER_ID:
        return
    
    allow_all_enabled = True
    update.message.reply_text("Now all group members can use the Like command once.")

def stop_all(update: Update, context: CallbackContext):
    global allow_all_enabled
    if update.effective_user.id != OWNER_ID:
        return
    
    allow_all_enabled = False
    update.message.reply_text("Permission for all has been disabled.")

def like(update: Update, context: CallbackContext):
    global allow_all_enabled, GROUP_IDS, saved_message

    if update.effective_chat.id not in GROUP_IDS:
        update.message.reply_text("This command does not work for this group.")
        return
    
    user_id = update.effective_user.id
    if user_id != OWNER_ID:
        if allowed_users.get(user_id, 0) <= 0:
            if not allow_all_enabled or (allow_all_enabled and allowed_users.get(user_id, None) is not None):
                update.message.reply_text("You have used up your limited requests for today.\n\nFor unlimited requests, contact: @taa1aibek11")
                return
            allowed_users[user_id] = 0

    try:
        region = context.args[0]
        player_id = context.args[1]
    except IndexError:
        update.message.reply_text("Please provide a valid region and UID. Example: /like sg 10000001")
        return  

    wait_message = update.message.reply_text("Sending MAX Likes Please wait...")

    url = f"https://likes.freefireinfo.site/api/{region}/{player_id}?key=Taalaibeklm4"
    response = requests.get(url).json()

    context.bot.delete_message(chat_id=update.message.chat_id, message_id=wait_message.message_id)

    if response.get('status') == 1:
        data = response['response']
        key_remaining = data['KeyRemainingRequests']
        remaining_requests = int(key_remaining.split('/')[0])
        
        # If there are no remaining requests
        if remaining_requests <= 0:
            update.message.reply_text("❗API request limit reached. Please wait until 1:30 (Sri Lankan time) before trying again.")
            return
        
        message = (f""
                   f"Name: {data['PlayerNickname']}\n"
                   
                                    
                   f"Before Likes: {data['LikesbeforeCommand']}\n"
                   f"After Likes: {data['LikesafterCommand']}\n"
                   f"Likes given by bot: {data['LikesGivenByAPI']}\n"
                   
                   f"Remaining requests: {key_remaining.split('/')[0]}/{key_remaining.split('/')[1]}\n\n"
                   f"Contact: @taa1aibek11\n"
                   f"{saved_message}")

        update.message.reply_text(message)

        if user_id != OWNER_ID and allowed_users.get(user_id) is not None:
            allowed_users[user_id] -= 1
    else:
        update.message.reply_text("Has already received Max Likes for Today. Please wait until 1:30 AM Sri Lankan time for the next request.")

def set_message(update: Update, context: CallbackContext):
    global saved_message
    if update.effective_user.id != OWNER_ID:
        return

    user_message = ' '.join(context.args)
    if not user_message:
        update.message.reply_text("Usage: /setmessage {your message}")
        return

    saved_message = user_message
    update.message.reply_text(f"Message saved: {saved_message}")

def send_welcome(update: Update, context: CallbackContext):
    if update.effective_user.id != OWNER_ID:
        return

    update.message.reply_text(
        "<b>All commands in the Like bot!\n"
        "/like {region} {UID} - This is a command to give 100 likes to a Free Fire account. Usage: /like sg 10000001\n"
        "/allow {user ID} {number of requests} - This command allows a Telegram user to use the Likes bot with a specified number of requests.\n"
        "/allowall - This command temporarily grants permission to all members in the group to use the Like command once.\n"
        "/stopall - This command removes the temporary permission for all members in the group.\n"
        "/setgroup {group ID} - Sets a specific group ID for the bot to operate in.\n"
        "/removegroup {group ID} - Removes a specified group ID from the allowed list.\n"
        "/setmessage {message} - Sets a custom message to display with Like results.\n\n"
        "Note: These commands are for admin only.</b>",
        parse_mode='HTML'
    
    )

def main():
    updater = Updater('7535701953:AAEU2ZFk5pCyJTCRxJDVns-pSSDx_t4JM6I', use_context=True)

    dp = updater.dispatcher

    dp.add_handler(CommandHandler("setgroup", set_group))
    dp.add_handler(CommandHandler("removegroup", remove_group))
    dp.add_handler(CommandHandler("allow", allow))
    dp.add_handler(CommandHandler("allowall", allow_all))
    dp.add_handler(CommandHandler("stopall", stop_all))
    dp.add_handler(CommandHandler("like", like))
    dp.add_handler(CommandHandler("setmessage", set_message))  # Команда үчүн setmessage
    dp.add_handler(CommandHandler("command", send_welcome))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
