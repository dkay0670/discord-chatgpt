
# ChatGPT Discord Bot
A Discord bot that allows users to interact with OpenAI's `gpt-3.5-turbo` large language model, which is the same one used for ChatGPT.

Check out [the changelog](/Changelog.md) to see what's changed!

## Using the bot
You can have a conversation with the bot by sending it a DM or pinging it in a server with a message. In DMs, the bot is only able to remember your previous interaction (question and answer), unlike ChatGPT, which is able to remember much further back than that.

Conversation history doesn't work by default in servers, but you can reply to any message in a channel with a ping to the bot, along with another message. This will use the message you replied to as context. This works in DMs as well, if you want to re-address a message that was sent a while ago.

### Slash commands
* `/help`: Outputs some help information.
* `/stats [user]`: Outputs usage statistics. If a user is provided, their usage stats will be outputted instead.
* `/purge`: Purges all of your message entries from the database.

### Owner-only commands
* `/fullpurge`: Purges all message entries from the database.
* `/users`: Manages bot access.
    * `/users allow <user>`: Allows a user to use the bot.
    * `/users block <user>`: Blocks a user from using the bot.
    * `/users unset <user>`: Removes a user from the allow/block list.
    * `/users wipe`: Wipes the allow/block list.
    * `/users list`: Displays the list of allowed and blocked users.

## Running the bot yourself
1. [Download and install Node.js](https://nodejs.org/en/download/) if you don't have it
1. [Download and install SQLite](https://www.sqlite.org/download.html) if you don't have it
1. Clone (or download and unzip) the repository and `cd` into it with your terminal
1. Run `npm install`
1. Rename `config-template.json` to `config.json` and open it
1. [Generate an OpenAI secret key](https://platform.openai.com/account/api-keys) and paste it in the `openai.secret` config field
    * Note: Use of the Chat API isn't free. At the time of writing, it costs $0.002 for every 1000 [text tokens](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them). The token count of a request includes both the user's input, the model's output, and any conversational context provided.
1. [Create a new Discord application](https://discord.com/developers/applications)
    1. Set its name, description, and picture
    1. Copy the Application ID and paste it in the `discord.id` config field
    1. Go to the "Bot" tab and create a new bot
    1. Copy the bot token and paste it in the `discord.token` config field
    1. Scroll down and make sure "Message content intent" is on
1. Set your Discord user ID in the `discord.owner_id` config field.
1. Make any other changes to the config file ([see below](#configuration)), then save it.
1. Create the message database by running `sqlite3 main.db ".read schema.sql".`
    * This is a required step. See [Database](#database) for details.
1. Register the bot's slash commands by running `node registerCommands.js`
1. Start the bot with `node bot.js`
    * If you're on a Unix operating system, run `sh bot.sh` to start the bot and auto-restart it if it crashes.
1. Once the bot logs in, an invite URL will be logged. Open it and follow the instructions to add the bot to your server.
1. Try it out by DMing or pinging the bot with a question!

As the owner, you're always allowed to use the bot, but with `config.public_usage` disabled, nobody else will be able to. If a disallowed user tries using the bot, you'll get a DM with a button to allow them. You can also add users manually with the `/users add` command.

## Configuration
The main configuration is located in `config.json`:

* `openai`
    * `secret`: Your OpenAI secret key.
* `discord`
    * `id`: Your Discord application's ID.
    * `token`: Your Discord bot's token.
    * `owner_id`: Your Discord user ID. Users will be told to contact this user if they aren't allowed to use the bot.
    * `status`:
        * `type`: The status prefix. Can be `Playing`, `Watching`, or `Listening`.
        * `text`: The status text.
* `system_prompt`: A set of message objects to send with every request to the language model, before the user's input or context. These messages can be used to influence the behaviour of the model.
 
 Messages with a `role` of `system` are meant to instruct the model, and messages with `role`s of `assistant` or `user` can be used to start a conversation without the user's input.
 
 There are a handful of placeholders you can use to customize the message for each interaction:
    * `{bot_username}`: The bot's username
    * `{user_username}`: The user's username
    * `{user_nickname}`: The user's server nickname, or their username otherwise
* `max_input_tokens`: The max size of input messages in [text tokens](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them)
* `max_output_tokens`: The max size of the language model's responses in text tokens
* `usd_per_token`: The cost, in US dollars, of a single text token. This is used to calculate the cost values shown in the `/stats` command.
* `delete_message_days`: Message entries older than this number of days will be automatically deleted from the database.
* `public_usage`: If `true`, all users will be able to use the bot by default. If `false`, only users allowed with `/users allow` will be able to use the bot.
* `ignore_prefixes`: If one of these strings is present at the beginning of a message, the message will be ignored.
* `show_regenerate_button`: If `true`, a "Regenerate" button will be shown on all language model response messages. Responses can also be regenerated from the "Apps" context menu regardless of this setting.
* `http_server`
    * `enabled`: Whether or not to enable the HTTP server. Requests to `/invite` will redirect to the bot's invite URL, and all other requests will redirect to this GiHub repository.
    * `port`: The port to host the server on, if enabled.

## Database
The bot stores every message with its accompanying response in the database we generated during setup.

The database contains a single table, `messages`, with the following columns:

* `time_created`: The timestamp of the interaction
* `user_id`: The ID of the user who sent the message
* `channel_id`: The ID of the channel the message was sent in
* `input_msg_id`: The ID of the user's message
* `output_msg_id`: The ID of the response message
* `input`: The user's input
* `output`: The language model's response
* `count_tokens`: The total number of text tokens that this interaction used

This data is used for conversation history and regenerating responses. See the `delete_message_days` config option to set up automatic message deletion, and the `/fullpurge` slash command to safely wipe all saved messages.