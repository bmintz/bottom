lightweight asyncio IRC client
==============================

With a very small API, bottom_ lets you wrap an IRC connection and handle
events how you want.  There are no assumptions about reconnecting, rate
limiting, or even when to respond to PINGs.

Explicit is better than implicit: no magic importing or naming to remember for
plugins.  `Extend <user/extension.html>`_ the client with the same ``@on``
decorator.

----

Create an instance:

.. code-block:: python

    import asyncio
    import bottom

    host = 'chat.freenode.net'
    port = 6697
    ssl = True

    NICK = "bottom-bot"
    CHANNEL = "#bottom-dev"

    bot = bottom.Client(host=host, port=port, ssl=ssl)


Send nick/user/join when connection is established:

.. code-block:: python

    @bot.on('CLIENT_CONNECT')
    async def connect(**kwargs):
        await bot.send('NICK', nick=NICK)
        await bot.send('USER', user=NICK,
                       realname='https://github.com/numberoverzero/bottom')

        # Don't try to join channels until the server has
        # sent the MOTD, or signaled that there's no MOTD.
        done, pending = await asyncio.wait(
            [bot.wait("RPL_ENDOFMOTD"),
             bot.wait("ERR_NOMOTD")],
            loop=bot.loop,
            return_when=asyncio.FIRST_COMPLETED
        )

        # Cancel whichever waiter's event didn't come in.
        for future in pending:
            future.cancel()

        await bot.send('JOIN', channel=CHANNEL)


Respond to ping:

.. code-block:: python

    @bot.on('PING')
    async def keepalive(message, **kwargs):
        await bot.send('PONG', message=message)


Echo messages (channel and direct messages):

.. code-block:: python

    @bot.on('PRIVMSG')
    async def message(nick, target, message, **kwargs):
        """ Echo all messages """

        # Don't echo ourselves
        if nick == NICK:
            return
        # Respond directly to direct messages
        if target == NICK:
            await bot.send("PRIVMSG", target=nick, message=message)
        # Channel message
        else:
            await bot.send("PRIVMSG", target=target, message=message)


Connect and run the bot forever:

.. code-block:: python

    bot.loop.create_task(bot.connect())
    bot.loop.run_forever()


.. toctree::
    :hidden:
    :maxdepth: 2

    user/installation
    user/async
    user/api
    user/events
    user/commands
    user/extension
    dev/development

.. _bottom: https://github.com/numberoverzero/bottom
