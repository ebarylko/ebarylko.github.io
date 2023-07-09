{:title "Setting up the project part 1"
 :layout :post
 :tags  ["cljs"]
 :toc true}



## Setting up the template

To set up the skeleton of the project I used the day8/re-frame-template library, and I ran the command ` lein new re-frame haggadah +garden +kondo +github-actions +cider +10x +re-frisk`. This generated a base for my application, including the libraries listed in the `+name` portion of the command. For more information on each of these libraries, click [here](https://github.com/day8/re-frame-template). 

## Setting up firebase

### Creating the project in the console
I went to the [firebase](https://firebase.google.com/?gad=1&gclid=CjwKCAjwjYKjBhB5EiwAiFdSfs8e-1KZ06y9k_AJ2BfhLW42yqwPlUnheELNS498hf9ksVyTbV9pVxoCJmUQAvD_BwE&gclsrc=aw.ds) website and clicked on create a new project. After naming the project and agreeing to the use of google analytics, I was at the home screen with the details of my project in front of me. 

![firebase homscreen](/img/firebase_homescreen.png)

###  Setting up the core.cljs to initialize application
I then went to the terminal and ran `npm install firebase`. After that, I added a directory fb in my src directory, and created a file called config.cljs. In this file I added the configuration settings for my application, which can be found in the lower portion of the project settings.


```clojure 
(def firebase {
:apiKey "Api key value"
:authDomain "your domain"                                                                                        :databaseURL "your url"
:projectId "name of project"
:storageBucket "storageBucket details"
:messagingSenderId "Id details"
:appId "appId values",
:measurementId "measurementId value"
})
``` 


After that I created a file in src/haggadah called routes.cljs. I then added my dependencies so my name space declaration looked like this.
```clojure
(ns ^:dev/always haggadah.routes
  (:require
   [reitit.frontend.easy :as rfe]
   [reitit.frontend.controllers :as rfc]
   [reitit.coercion.spec :as rss]
   [reitit.frontend :as rf]
   [reagent.core :as r]
   [re-frame.core :as re-frame]
   [haggadah.views :as views]
   [haggadah.events :as events]))
```

With that I defined the routes for the application.
```clojure 
(def routes
  [
   ["/" {:name      :home
        :view      views/main-panel
        :link-text "Home"}]])
```
Then, I added the code below which initializes the routing, manages the logic for navigating to new pages, and displays the view corresponding to the current route.

```clojure
(defn on-navigate [new-match]
  (when new-match
    :current-route
    (re-frame/dispatch [:navigated new-match])))

;; This event handler changes what the current route should do when the user enters it.
(re-frame/reg-event-db
 :navigated
 (fn [db [_ new-match]]
   (let [old-match   (:current-route db)
         controllers (rfc/apply-controllers (:controllers old-match) new-match)]
     (assoc db :current-route (assoc new-match :controllers controllers)))))


;; this effect changes the current page to the one corresponding to [route]
(re-frame/reg-fx
 :push-state
 (fn [route]
   (apply rfe/push-state route)))

;; This subscription grabs the current route from the database
(re-frame/reg-sub
 :current-route
 (fn [db]
   (:current-route db)))

(def router
  (rf/router
   routes
   {:data {:coercion rss/coercion}}))

(defonce history (atom nil))

(defn init-routes! []
  (js/console.log "initializing routes")
  (rfe/start!
   router
   on-navigate
   {:use-fragment true}))

;; this function grabs the view corresponding to the current route and renders it properly on the page
(defn router-component [{:keys [router]}]
  (let [current-route @(re-frame/subscribe [:current-route])
        view (case (-> current-route :data :name)
               :home  views/main-panel
               views/main-panel)]
    [:div.main-container.is-flex.is-flex-direction-column
     [views/top-menu {:router router :current-route current-route}]
     (when current-route
       [:div.is-flex-grow-1.is-flex
        [view]])]))
```


With that done, I went into my core.cljs file located in src/haggadah, and made a few changes. First, I added firebase/app , fb.config , reagent.core, fb.firestore, fb.functions, fb.auth, react-dom/client, and fb.routes to my dependencies. It should look like this.

``` clojure
(ns haggadah.core
  (:require
   ["firebase/app" :as fba]
   ["react-dom/client" :refer [createRoot]]
   [haggadah.config :as config]
   [haggadah.events :as events]
   [haggadah.fb.auth :as fb-auth]
   [haggadah.fb.config :as cfg]
   [haggadah.fb.firestore :as fb-fs]
   [haggadah.fb.functions :as fb-fn]
   [haggadah.routes :as routes]
   [re-frame.core :as re-frame]
   [reagent.core :as r]))
```

With this in place, I was able to initialize the firebase instance by creating my `fb-init` and `firebase-init!` function. My fb-init function checks if there is an instance running, and if not, creates one with the specific configurations. My firebase-init! function does that as well, but is used with in the init function to set up the application.

``` clojure
(defonce firebase-instance (atom nil))

(defn dev-setup []
  (when config/debug?
    (println "dev mode")))

;; this function creates the container in which react components get rendered 
(defonce root (createRoot (.getElementById js/document "app")))

(defn ^:dev/after-load mount-root []
  (re-frame/clear-subscription-cache!)
  (.render root (r/as-element [routes/router-component {:router routes/router}])))


;; this function initializes firebase and also connects the authentication, firestore, and functions emulators
(defn fb-init [config]
  (when-not @firebase-instance
    (let [cfg (clj->js config)]
      (reset! firebase-instance (fba/initializeApp cfg))
      (fb-auth/init @firebase-instance events/auth-user-success)
      (fb-fs/init @firebase-instance)
      (fb-fn/init @firebase-instance))))

(defn firebase-init!
  []
  (fb-init cfg/firebase))

(defn init []
  (re-frame/dispatch-sync [::events/initialize-db])
  (dev-setup)
  (mount-root)
  (firebase-init!)
  (routes/init-routes!))
```


### Setting up the firebase CLI
The firebase CLI gives users a way of using firebase commands from the terminal. To use this, I first ran `curl -sL https://firebase.tools | bash`. After, I ran `firebase login` so that I would be verified and given permission to use the CLI. 

### Setting up the features of the application
Run `firebase init` within the app directory so that you can configure your application. Select all the features except Database. You'll then be asked to select a Firebase project to connect to your local project directory. Select existing project, and select the project with the name you assigned earlier when creating the project in the console. It'll then ask for the name of your public folder, just accept it as public. When asked if you want to setup automatic builds and deploys with github, select yes. Click yes on having your application be SPA (single page application).

### Setting up the firebase emulators
The emulators allow you to see how your project runs locally, allowing you to review and test your site before deploying it. Run `firebase init emulators`. Pick all emulators except Database, and accept default port locations. 

### Other small details to fix
Head to the package.json file, and delete the "functions" key and its value. Furthermore, in the hosting key, change the value of the public key to "resources/public".

## Setting up automated testing

### Editing the karma.conf.js
We first want to intall Karma, which will allow us to run integration tests. run `npm install karma --save-dev`. Open your karma.conf.js, and make the follwing changes:
* Delete the junitOutputDir var
* For the value of :browsers in config, change it to 'ChromeHeadlessNoSandbox'
* Change the value of :files within config from 'karma-test.js' to 'ci.js'
* Delete the junitReporter default configuration
* Add the key value pair `singleRun: true`to client
* Add the customLaunchers key after client, adding the settings shown in the photo. 

Your file should look like this.


``` clojure
module.exports = function (config) {
 
    config.set({
        browsers: ['ChromeHeadlessNoSandbox'],
      basePath: 'target',
      files: ['ci.js'],
      frameworks: ['cljs-test'],
      plugins: [
          'karma-cljs-test',
          'karma-chrome-launcher',
      ],
      colors: true,
      logLevel: config.LOG_INFO,
      client: {
          args: ['shadow.test.karma.init'],
          singleRun: true
      },
        customLaunchers: {ChromeHeadlessNoSandbox:  {'base': 'ChromeHeadless', 'flags': ['--no-sandbox']}},
 
    })
  }

```



### Adding build scripts
Head to the shadow-cljs.edn file and go to the builds in :builds. Add a new build called :ci, with the values shown in this photo.


``` clojure 
 :ci {:target :karma
      :output-to "target/ci.js"}

```


Now head to the package.json file, and add a "test" key to scripts. It should have this appearance. 

``` clojure
 "test": "npx shadow-cljs compile ci && npx karma start --single-run""
```


Now head to the package.json file, and add a "test" key to scripts. It should have this appearance. 


### Editing the workflows
The workflows are files that run a series of commands everytime a specific github command is detected. For example, a workflow could be configured to run the tests for your project everytime a pull request is detected. 

There are three workflows we need to edit, one that deploys to firebase when merging a branch, one that runs the tests for the project when pull/push requests are detected, and one that deploys to firebase on pull requests.

Head to the .github/workflows/firebase-hosting-merge.yml file. Under steps, where it says run, change the command associated with run to `npm install && npm run release`.

Now go to the .github/workflows/firebase-hosting-pull-request.yml file. Under steps, where it says run, change the command associated with run to `npm install && npm run release`.

Lastly, go to .github/workflows/test.yaml, and change it so it looks exactly like this. 


``` clojure 
name: Run tests
 
on: [push, pull_request]

jobs:
  test:
    name: Tests
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/day8/chrome-latest:5.0.0
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3 #Setup Node
        with:
          node-version: '18'
      - name: run test
        run: |
          npm install
          npm test
```
=======

## End of part one

This is the end of the first part of setting up the project. In the next post we'll finish setting everything up and then we can start adding features.

