# Custom FTC Dashboard Write-up
Write-up/Guide on how to customize the [acmerobotics FTC Dashboard](https://github.com/acmerobotics) without using new libraries or importing code onto the control hub

Concepts covered in this write-up are meant for REV Control Hubs

**DISCLAIMER:** SOME OF THE CONCEPTS SHOWN HERE HAVE NOT BEEN TESTED, THEY ARE THEORETICAL AND SHOULD WORK(I'm too lazy to test them)

# Why Customize Your Dashboard?
Most teams won't need this because the default dashboard is enough, however, sometimes a little customization could go a long way in boosting productivity. Unfortunately(unless I'm stupid), there is no simple way to modify the dashboard and directly compile it onto the bot. This write-up will explain a simple(or complex) workaround to customize the dashboard and do whatever you want with it.

# 1. The Concept
There is nothing special about this, it's just a simple data relaying method to a self-hosted dashboard. A simple example would go as follows: Bot transmits data to a server/backend on a computer which then relays the data to the self-hosted dashboard. This has been tested using websockets, but there are other ways to go about transmitting the data. 

# 2. A Faster Way
A faster way to do the same thing done in #1 is to just directly relay all the data from the bot to the dashboard. However, this hasn't been tested yet since it's very hard to debug when it's end to end and hosting the websocket server on either the dashboard or the bot is generally not a good idea.

# 3. Actually Customizing the Dashboard
To customize the dashboard, it's pretty simple. Download the dashboard source code from [here](https://github.com/acmerobotics/ftc-dashboard)<br> Follow the installation instructions in the README.md to get started. All the code for the actual dashboard is located in the client folder. To customize the site, simply go through the different components of the page and change what you want. You should be able to add/remove components, but I have only tested changing the content due to my limited knowledge of React. You can preview your site by going into the directory and running `yarn dev` After running the command, Vite will create a page on your machine where you can make changes and view them in real time. 

# 4. Deploying and Running
To deploy and run, you will need 3 components(or 2 if you're using end to end) <br>
Components: <br>
- Script on the bot that can connect to websockets
- A locally hosted websocket server that serves as a median
- Custom dashboard code deployed<br>


In order to run the script on the bot, have it embedded or called within a teleop or auto(not tested) class. To run the websocket server, run a simple handler while connected to the bot wifi and bind the websocket to the bots IP '192.168.43.62:PORT' To run the dashboard code, cd into the directory containing the code for the dashboard and run 'yarn dev --host' and now everything setup on the bots wifi. 

## This is still WIP
