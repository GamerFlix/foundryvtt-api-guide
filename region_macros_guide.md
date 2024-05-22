# Introduction

This guide is currently under construction and should not be relied upon

This guide will walk you through how to make your Macros or Scripts work with the Scene region behaviours of "Execute Macro" or "Execute Script" respectively. In doing so it however assumes that you already know how to accomplish the relevant task in a normal Macro or Script. If this is not the case please refer to [this guide which introduces you to basic Foundry infrastructure](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md)

For general information on Regions refer to the Official Knowledge Base article on Regions TO BE ADDED or this video tutorial by alaustin here TO BE ADDED

# Setting up your Region

Before you can execute a Macro or Script with a Region you have to create one and add the "Execute Macro" or "Execute Script" Behavior to it under the Behaviors Tab.

IMAGE1

From there you will be greeted with one of the following two windows.

IMAGE2

In the case of the "Execute Script" behavior you can input the relevant code directly into the Region config while for the "Execute Macro" behaviour you have to specify which Macro should be executed.

Don't forget that you need to actually have something to trigger the behavior which you configure under the 