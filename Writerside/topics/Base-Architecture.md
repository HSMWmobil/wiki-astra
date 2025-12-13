# Base Architecture

The app follows the base idea of <tooltip term="MVVM">MVVM</tooltip>, adhering to the [official 
Flutter architecture guideline](https://docs.flutter.dev/app-architecture). However, some 
adjustments are established in terms of terminology, further explained in this page.
We also provide a few samples on how to follow our guidelines in order to establish new or 
extend existing features of the app.

## MVVM Adaption

<tooltip term="MVVM">MVVM</tooltip> is adapted by using the term of **controllers** for the 
centralized logical unit instead of _view model_. The relation is as follows and also expressed 
by the diagram below:
 - The **view** is the visual element accessed by the user, therefore representing the actual 
screen which is accessed within the app. It should not contain any complex logic (exceptions are 
conditional renderings of images or animations). Interacting with the screen may trigger 
operations within the controller.
 - The **controller** (in context of <tooltip term="MVVM">MVVM</tooltip>: the _view model_) 
accepts incoming triggers of the UI and handles logical operations internally. It is responsible 
for notifying the UI when changes that need to be rendered happen and hosts all operations to 
load model data.
 - **Models** define data structures which need to be represented within the UI. This can be 
complex data models fetched by an API service or just some primitive data types needed for the 
current screen. For the latter, it is not always needed to put models into separate classes. 
Instead, variables may just be established and maintained within the controller itself.

![Alt Text](app/img/base-mvvm.png){ width="500" }


## MVVM Implementation in Flutter


