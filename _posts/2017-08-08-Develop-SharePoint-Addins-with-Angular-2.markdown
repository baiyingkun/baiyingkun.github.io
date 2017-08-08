---
layout: post
title: "Develop SharePoint Online Add-ins with Angular 2 and WebPack"
date: 2017-08-08 14:02:00 +0200
categories: tech
---


## Overview ##
The aim of this blog is a study note about how to develop a SharePoint Online Add-in with Angular 2 (4.3.3) and WebPack. It's based on the tutorial from: [Original post](https://amalzblog.wordpress.com/2016/05/30/develop-sharepoint-addin-with-angular-2/). After one year later since this post, there're a lot of upgrades for both [Angular 2](https://angular.io/) as well as [WebPack](https://webpack.github.io/) (from 1.X to 2.X). I note down the modifications I have made to make things work. 

The approach makes use of the official Angular2 webpack seed in: [Angular-Webpack Introduction](https://angular.io/guide/webpack). The result is to embedding this seed as an Add-in inside SharePoint. 

> 1. The approach uses **VS** (2017) + **TypeScript** with **NPM**, **WebPack** (2.2.1) and **Gulp** tools. 
>2.  It also assumes you have already a Office 365 Developer site to develop, test and deploy Office and SharePoint Add-ins. (You could start with a free 30-day trial with one user license, refer here: [How to Sign up for an Office 365 Developer Site](https://dev.office.com/sharepoint/docs/sp-add-ins/set-up-a-development-environment-for-sharepoint-add-ins-on-office-365#sign-up-for-an-office-365-developer-site) )
> 3. The time when I'm writing this blog, Angular is under version 4.3.3

## Setup Project Structure ##

###  1. Create SharePoint Hosted Add-in Project###
Create a new project in VS with the template of "SharePoint Add-in". Note that, you need to provide the SharePoint site in the setting step. Also select *SharePoint-hosted*  for hosting our Add-in. 
![Create Add-in in VS]({{site.url}}/assets/create-add-in.png)

![Setting of the Add-in]({{site.url}}/assets/add-in-settings.png)


### 2. Add 'app' Module###
We are going to contain all our client side artifacts in a SharePoint module called ‘app’. Just use “Add New Item -> Module” and add a module.


### 3. Remove unnecessary Modules###
Remove the default modules that wre're not going to use.

 1. Move 'Default.aspx' from 'Pages' to 'app'. Then delete 'Pages' module
 2. Delete 'Content' module


## Embed WebPack demo inside the Add-in##
First download the demo application through [Angular-Webpack Introduction](https://angular.io/guide/webpack). You will have a folder with the files below:

![structure of webpack demo]({{site.url}}/assets/webpack-angular-demo.png)

### 1. Copy all the files inside the project###
### 2. Install Packages###
Open a command prompt and change the directory to the VS project path. Note that we need to change to the folder of the project files, not the folder containing '.sln'. Usually for the projects of VS, it's the folder in the same level of the solution.

 Use below command to install all the packages according to 'package.json'. This may take a while. 

    npm install

After the installation, you could see a similar result inthe command prompt:
![Npm-nodes-installed]({{site.url}}/assets/npm-nodes.png)
And if you open your project folder in file explorer, you could see a folder named node_modules. It contains all the dependencies of Angular application.

### 3. Setup WebPack to generate Bundles
If you take a look inside “node_modules” folder, you will notice that there are number of dependancies for Angular2 to function properly. We cannot expose all these in our Addin. Thats where webpack comes to rescue.

Webpack takes modules with dependencies and generates static assets representing those modules. It analyses the “import” statements and generate modules based on a configuration that we specify. We simply use webpack to generate bundles of client side dependancies for our Angular2 application to work properly.

Idea is to generate below bundles,

 - **polyfills** : Bundle all polyfills related Javascript to a single bundle.
   
 - **vendor**: Bundle all third party Javascript (including Angular) to a single bundle. 
   
 - **app** : Bundle all Javascript produced by
   transpiling the TypeScript that we write.
   
#### 3.1 Modify 'config/webpack.dev.js'####
We need some modifications of the webpack configuration file to make the bundle be generated inside module 'app'.

#### 3.2 Modify 'config/webpack.common.js'####
We need to remove the 'HtmlWebpackPlugin' used in the demo app. This plugin will generate a single index.html file including all the webpack bundles. 

Inside this file, remove all the codes related with "HtmlWebpackPlugin".

### 4. Install WebPack###
Install Webpack globally. So you can run the command to generate the bundles on from Command Line.

    npm install -g webpack

### 5. Run WebPack to generate Bundles ###
Run webpack to generate the bundles. This command will output the bundles in ‘\app’ directory.

    webpack --config .\webpack.config.js --progress --colors

![Run webpack]({{site.url}}/assets/run-webpack.png)

### 6. Include generated bundles in the project###
After running webpack, we have serveral generated files and we need to add them inside the project. 
![Include in the project]({{site.url}}/assets/include-in-project.png)

### 7. Modily "app/Default.aspx" to be the entry point for angular app.
Now that we have generated the bundles; we can refer them in our pages. Add the references to bundles in ‘\app\Default.aspx’.

{% highlight xml linenos %}
<asp:Content ContentPlaceHolderID="PlaceHolderAdditionalPageHead" runat="server">
    <SharePoint:ScriptLink name="sp.js" runat="server" OnDemand="true" LoadAfterUI="true" Localizable="false" />
    <meta name="WebPartPageExpansion" content="full" />
    <link href="/angular-webpack-addin/app/app.css" rel="stylesheet">
</asp:Content>    
<asp:Content ContentPlaceHolderID="PlaceHolderPageTitleInTitleArea" runat="server">Page Title
</asp:Content>    
<asp:Content ContentPlaceHolderID="PlaceHolderMain" runat="server">
    <my-app>Loading...</my-app>
    <script type="text/javascript" src="/angular-webpack-addin/app/polyfills.js"></script>
    <script type="text/javascript" src="/angular-webpack-addin/app/vendor.js"></script>
    <script type="text/javascript" src="/angular-webpack-addin/app/app.js"></script>
</asp:Content>
{% endhighlight %}

### Run the app ###
Press F5 to make the add-in running and connecting to sharepoint. You should see the result like below: 
![Run app from SP]({{site.url}}/assets/run-webpack-from-sp.png)


 
 




 
 

