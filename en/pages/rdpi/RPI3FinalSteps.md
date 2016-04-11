# Final Steps
Final steps that should be completed to have a fully functional RADIUSDesk WiFi hot-spot.

## Passwords should be changed (Do this!)
There are a number of passwords that should be changed before we can make this hot-spot publicly accessible.

1. The *sudo* password of the Raspberry Pi should be changed. Currectly it is `raspberry`, with the only user being `pi`. This is as per the default setup for Raspbian.
  * Login to the pi via __ssh__ or __tty__.
  * Type `passwd`
  * Follow the prompts (default pass is `raspberry`)
2. The RADIUSDesk password __*TODO*__
3. __*TODO*__ Possibly others?

## Add details to RADIUSDesk (Do this!)
This section is not very exhaustive. There is much info on the main RADIUSDesk site, and a few videos too.

> W. I. P.

## Use PPPOE to initiate connection (Optional)
If you have a modem and the Pi is the only device connected to the Internet, then you could use the Pi to initiate the ADSL PPPOE connection. This enables the Pi to have a little more control over the Internet connection, and removes some of the modems "optimizations"/fiddlery that sometimes makes things... weird to deal with. It also has the added benefit that you can secure the connection a little better, as modems tend to have exploits and back-doors, either built in or discovered. Though, you have to __DO THIS PROPERLY__ if you are going to do it at all, and expect it to do a better job than your router.

__WARNING!__ If you do this step:
  1. You have to setup a firewall,
  2. Change your password to something very secure,
  3. Otherwise disable passwords completely and use public/private key pairs
  4. And maybe change the `SSH` port. 
  Otherwise, it probably won't be long before your little Pi becomes a slave in a bot-net.

> W. I. P.

## Install and setup a firewall (Optional but recommended)

> W. I. P.

## Install QOS systems (Optional)

> W. I. P.
