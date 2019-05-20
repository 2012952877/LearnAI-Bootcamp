# Building Intelligent Bots with Microsoft's Bot Framework (.NET)

This hands-on lab guides you through creating an intelligent bot from end-to-end using the Microsoft Bot Framework, Azure Search, and Microsoft's Language Understanding Intelligent Service (LUIS).

## Important Information - May 2019

This Lab 2.2 is provided “as is”, including steps 1 to 5. It will work with Bot Framework SDK V4.0 but not the Echo-bot template (2019), SDK V4.3 and its newer cognitive services integration methods. Security and resources management are done from a .bot file (SDK V4.3); later SDK versions will want to use appsettings.json or .env file. 

**You are invited to add these updates to the repo, including new cognitive services integrations like Q&A Maker. Pull Requests are reviewed monthly and credit appropriately attributed.**

## Objectives

In this workshop, you will:

- Understand how to implement Azure Search features to provide a positive search experience inside applications
- Build an intelligent bot (with C#) using Microsoft Bot Framework (SDK v4) that leverages LUIS and Azure Search
- Use Middleware to make bots more efficient and optimize calls to LUIS.

While there is a focus on LUIS and Azure Search, you will also leverage the following technologies:

- Data Science Virtual Machine (DSVM)
- Visual Studio

## Prerequisites

This workshop is meant for an AI Developer on Azure. Since this is a short workshop, there are certain things you need before you arrive.

Firstly, you should have experience with Visual Studio. We will be using it for everything we are building in the workshop, so you should be familiar with [how to use it](https://docs.microsoft.com/en-us/visualstudio/ide/visual-studio-ide) to create applications. Additionally, this is not a class where we teach you how to code or develop applications. We assume you know how to code in C# (you can learn [here](https://mva.microsoft.com/en-us/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)), but you do not know how to implement advanced Search and NLP (natural language processing) solutions.

Secondly, you should have some experience developing bots with Microsoft's Bot Framework. We won't spend a lot of time discussing how to design them or how dialogs work. If you are not familiar with the Bot Framework, you should complete [this tutorial](https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) prior to attending the workshop.

Thirdly, you should have experience with the portal and be able to create resources (and spend money) on Azure. We will not be providing Azure passes for this workshop.

>Note: This workshop was developed and tested with Visual Studio Community 2017

## Introduction

We're going to build an end-to-end scenario that allows you to pull in your own pictures, use Cognitive Services to find objects and people in the images, figure out how those people are feeling, and store all of that data into a NoSQL Store (CosmosDB). We'll use that NoSQL Store to populate an Azure Search index, and then build a Bot Framework bot using LUIS to allow easy, targeted querying.

> Note: This lab combines some of the results obtained from various labs (Computer Vision, Azure Search, and LUIS) from earlier in this workshop. If you did not complete the above listed labs, you will need to complete the Computer Vision, Azure Search, and LUIS labs before moving forwards. Alternatively, you can request to use a neighbor's keys from their Azure Search/LUIS labs.

## Architecture

In a previous lab (`lab01.1-computer_vision`), we built a simple C# application that allows you to ingest pictures from your local drive, then invoke the [Computer Vision](https://www.microsoft.com/cognitive-services/en-us/computer-vision-api) Cognitive Service to grab tags and a description for those images.

Once we had these data, we processed it and stored all the information needed in [CosmosDB](https://azure.microsoft.com/en-us/services/documentdb/), our [NoSQL](https://en.wikipedia.org/wiki/NoSQL) [PaaS](https://azure.microsoft.com/en-us/overview/what-is-paas/) offering.

Once we had the data in CosmosDB, we built an [Azure Search](https://azure.microsoft.com/en-us/services/search/) Index on top of it (`lab02.1-azure_search`). Next, we will build a [Bot Framework](https://dev.botframework.com/) bot to query it. We'll also extend this bot with [LUIS](https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis) to automatically derive intent from your queries and use those to direct your searches intelligently.

![Architecture Diagram](./resources/assets/AI_Immersion_Arch.png)

> This lab was modified from this [Cognitive Services Tutorial](https://github.com/noodlefrenzy/CognitiveServicesTutorial).

### Navigating the GitHub

There are several directories in the [resources](./resources) folder:

- **assets**, **instructor**: You can ignore these folders for the purposes of this lab.
- **code**: In here, there are several directories that we will use:
  - **Models**: These classes will be used when we add search to our PictureBot.
  - **FinishedPictureBot-Part0**: The finished PictureBot.sln that is a simple "Hello World" bot. If you fall behind or get stuck, you can refer to this.
  - **FinishedPictureBot-Part1**: The finished PictureBot.sln that includes additions for Regex. If you fall behind or get stuck, you can refer to this.
  - **FinishedPictureBot-Part2**: The finished PictureBot.sln that includes additions for Regex and Search. If you fall behind or get stuck, you can refer to this.
  - **FinishedPictureBot-Part3**: The finished PictureBot.sln that includes additions for Regex, LUIS and Azure Search. If you fall behind or get stuck, you can refer to this.

> **Note**: If you decide to use a finished solution, you will still need to add the keys to connect to services, and **you will not be able to publish the bot**. We recommend comparing the finished solution to your solution and making adjustments to your solution as needed (as opposed to just opening and running from the finished solutions).  
>
> You need Visual Studio to run these labs. If you have already deployed a Windows Data Science Virtual Machine for one of the labs, we recommend using that.

## Collecting the Keys

Over the course of this lab, we will collect various keys. It is recommended that you save all of them in a text file, so you can easily access them throughout the workshop. You may already have already started collecting these keys in previous labs, so you can keep on using the same text file.

>_Keys_
>
>- LUIS App ID:
>- LUIS Key:
>- LUIS URI:
>- Azure Search Name:
>- Azure Search Key:
>- botFilePath:
>- botFileSecret:

## Setting up for bot development - Bot Framework Emulator

We will be developing a bot using the latest .NET SDK (v4).  To get started, we'll need to download the Bot Framework Emulator, and we'll need to create a Web App Bot and get the source code.  

Download and install the Bot Framework Emulator for testing your bot locally. The instructions for the rest of the labs assume you've done this step. You can download the emulator by going to [this page](https://github.com/Microsoft/BotFramework-Emulator/releases) and downloading the most recent version of the emulator. 

The emulator installs to `c:\Users\`_your-username_`\AppData\Local\botframework\app-`_version_`\botframework-emulator.exe` or to your Downloads folder, depending on browser.  

**If you're using a DSVM, you'll need to run the Emulator as an administrator.**

Now that you've downloaded and open the Bot Emulator, the next thing you have to set up is [ngrok, which allows us to connect to bots hosted remotely](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0#configure-ngrok). While most of the testing we'll do is local, you'll need ngrok to access the published version of the bot in the Emulator. There are two main steps to set this up:

1. Download ngrok from this website: <https://ngrok.com/download.> Click "Save as" when you download it so you can control where it gets saved (pick your Documents or Downloads folder, it should default to Downloads). Next, navigate to the install location and unzip the file (right click and select "Extract all...") and extract it one level up from the downloaded file (e.g. to  C:\Users\antho\Downloads\).  
2. Open the Bot Framework Emulator and hit the settings button (gear icon in bottom left corner). In the "Path to ngrok" box, hit "Browse," find "ngrok.exe", click "Select ngrok", and then click "Save".

## Navigating the Labs

This workshop has been broken down into five sections:

- [1_Dialogs_and_Regex](./1_Dialogs_and_Regex.md): Here you will build a bot that uses Regex to act on user input and learn about using dialog containers for organizing bots.
- [2_Azure_Search](./2_Azure_Search.md): We'll configure our bot for Azure Search and connect it to the Azure Search service from the previous lab.
- [3_LUIS](./3_LUIS.md): Next, we'll incorporate our LUIS model into our bot, so that we can call LUIS when Regex does not recognize a user's intent.
- [4_Publish_and_Register](./4_Publish_and_Register.md): We'll finish by re-publishing and registering our bot.
- [5_Closing](./5_Closing.md): Here you'll find a summary of what you've done and where to learn more.

## Continue to [1_Dialogs_and_Regex](./1_Dialogs_and_Regex.md)