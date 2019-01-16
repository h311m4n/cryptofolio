# cryptofolio
Simple web based crypto portfolio manager &amp; start/stop mining rigs based on solar array output 

This web application has 2 purposes:

1. It allows you to have a web page to manage your crypto portfolio. I've tried a number of them over the years, yet could never find
a simple one that listed all the coins I have, like some very low marketcap ones. This webapp uses coingecko's site and API to build
your portfolio.

2. If you are a miner and have a solar array with a way to pull the watt output at every given moment & remote power plugs (zwave, wifi plugs etc.)

You can adjust my script to get your rigs to start and stop automatically based on the solar output your array is generating. No more
huge electricity bill!

I'm running this web application on a centos virtual machine with apache and mysql.

# What you need to manage your rigs:
As this project is really specific to my installation, here's what I use. You can probably adapt your code/hardware to make it work on your side.

1. A solar array, obviously, with an inverter from which you can fetch the output at any given moment. I have a StecaGrid 5503 which comes with an ethernet port. It has a webserver running on it, but its very basic. One of the webpages shows a whole lot of information like the solar output. So I simply use a json_encode to convert the page to a json array and then parse it to find the wattage/solar output of my grid which is the basis for my script to decide to turn a rig on or off

2. Install NRPE on your mining rigs (NSClient for Windows). For those not familiar with it, it's an agent used to report the status of a windows machine to a monitoring service like icinga for instance. You can use pretty much any sort of scripting language to pull information from your machine or execute commands on it remotely. I use it simply to issues a simple Windows "shutdown -s -f -t 30" on my rigs

3. A way to shut your rigs on and off physically and monitor their wattage. Since pretty much forever, you can set your bios to start a computer when power is cut off and on. For this, I use Aeotec Z-wave plugs and OpenHAB. By simply sending a REST API command to OpenHAB, a plug can but turned off and on and a rig will automatically boot up when it is off. These plugs also report the power usage.

4. A simple table in mysql with all your rigs and assign them a start sequence. For me, the first rig to start is the one that uses the less power all the way up to the one that uses the most power. I don't specify the base wattage in the database as these are values that can be fetched from OpenHAB for instance when your rig runs. So for instance you'll have:

RIG | SEQUENCE | Base Watts |WATTS (current) | STATUS (1 = on, 0 = off)
1       1            647              650      1
2       2            660              0        0
3       3            700              0        0
...

# How the script works
I'll be honest, its a very dumb script that runs using a cronjob every 10 minutes. The simplified run down is:

1. Get the solar output, for example 1400W
2. Check if there are rigs online already. In this case, lets assume RIG1 is online and uses 650 Watts
3. Calculate how many Watts are available to run a rig: 1400 - 650 = 750
4. Rig in sequence 1 is already started, checking base watts for rig in sequence 2 = 660
5. 660 < 750, so we can start rig in sequence 2 because the solar output is more than what this rig would use


