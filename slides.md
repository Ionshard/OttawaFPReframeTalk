![re-frame logo](img/re-frame_512w.png)
> "It's MVC, Jim, but not as we know it"
Note: Introduce re-frame which is a clojure libary for architecting react applications.

This is just a very short overview to spark interest

To quote McCoy



## Introduction
### Victor Ling
Software Developer At CENX



![React logo](img/react-logo.png)

A Javascript library for developing UIs by Facebook

Note: Works by allowing us to define our HTML from scratch every frame

Similar to the way a video game works

Doing that with pure DOM would be slow, React uses a VDOM and really smart diffing algorithms



## Reagent
#### Minimalistic React for ClojureScript
Efficient React components using nothing but plain ClojureScript functions and data

Note: If you have ever used React there is this odd JSX stuff



## Hiccup
* A way to represent HTML using Clojure Datastructures
* Used by Reagent to build React VDOM

Note: HTML is just a tree written in XML, so why not have a tree in Clojure?


## Example
````
[:div {:class "page-main"}
  [:h1 "Hello"]
  [:a {:href "www.google.com"} "Check out Google!"]]
```` 

becomes

````
<div class="page-main">
  <h1>Hello</h1>
  <a href="www.google.com">Check out Google!</a>
</div>
````



## Reactive Components
In Reagent we can make components that are smart enough to only rerender when their data has changed.


## Example
````
(def click-count (r/atom 0))

(defn counting-component []
  [:div
   "The atom " [:code "click-count"] " has value: " @click-count ". "
   [:input {:type "button" :value "Click me!"
            :on-click #(swap! click-count inc)}]])
````



## What we have so far?
Components &rarr; Hiccup &rarr; Reagent &rarr; VDOM &rarr; React &rarr; DOM



## So What is Re-frame?
* It is MVC with Pure Functions
* It uses derived data in an FRPish way
* That derived data only flows in one direction

Note: It is MVC but with pure functions rather than objects

It uses derived data in an Functional Reactive Programming ish way



## What about Reagent?

Reagent provides the `V` in re-frame's new `MVC`



## Okay how about the `M` & `C`?

Alright I will admit this is where the MVC metaphor breaks down.



## Flow
Flow is very important to re-frame and is implemented via Reagent's tools that allow reactive components

You have to think of your mutable state as a stream of data that changes over time



## Signal Graph
What we want to make is essentially a signal graph that allows our data to flow effectively to the components as that data changes



## The `app-db`
### An in-memory database
* The root of the signal graph
* All state is stored here
* Can be controversial unless we're talking about an actual database
* Normally implemented by a `ratom`

Note: This is where re-frame starts


## `ratom`
#### Reactive Atom
* An `atom` is Clojure's way to deal with mutable data
* A `ratom` is an atom that pays attention to who derefs (uses) it
* Provided by Reagent



## Okay, but How?
We have our data and our components, how do we connect them?

Ultimately, components should do a couple of things.


### 1. obtain data from `app-db`


### 2. ... while knowing as little about the structure of the `app-db`
Decoupling is a good thing


### 3. Be reactive
They should automatically recompute their hiccup as their data changes over time



## Subscriptions
Re-frame's solution to the above problems

* Subscription Handlers transform the `app-db` and return a `ratom`
* Components, or other subscriptions can then subscribe to these `ratoms`
* The components will automatically be rerendered when the data of the subscription changes over time.



## The Full Picture
### UI as a function of our state
`app-db` &rarr; Subscriptions &rarr; Components &rarr; ... &rarr; DOM

Note: We essentially have represented our DOM as a pure function of the app-db



## Now What?
How does `app-db` change over time?



## Events
#### The 2nd Flow
Events are how the `app-db` changes over time.

They are:
* Pure Data
* Dispatched


## Example
Events could be anything but usually they are just a simple vector like

````
[:yes-button-clicked]
````

These can include additional information like

````
[:delete-item 42]

````


## Real Example
````
(defn yes-button []
  [:div {:class "button-class"
         :on-click #(dispatch [:yes-button-clicked])}
        "Yes"])
````

Where `[:yes-button-clicked]` is the event.



## Handlers
Registered to handle Events

* Handled Asynchronously
* Pure functions
* Only thing that modifies the `app-db`


## Example
Here we write a simple function that takes the `app-db` and some parameter determining the item-id which we want to delete.

````
(defn handle-delete
  [app-db [_ item-id]]
  (dissoc-in app-db [:some :path item-id]))
  
(reg-event-db
  :delete-item
  handle-delete)
````



## The final result
##### First Flow
`app-db` &rarr; Subscriptions &rarr; Components &rarr; ... &rarr; DOM

##### Second Flow
`app-db` &larr; Event Handlers &larr; Dispatch Events &larr; DOM



## The endless cycle of derived data
From the re-frame README
>DDATWD - Derived Data All The Way Down



# The End
That's It

Questions?



## Subscription Example
````
(register-sub
 :sorted-items
 (fn [db [_]]
   (reaction
      (let [items      (get-in @db [:items])
            sort-attr  (get-in @db [:sort-by])]
          (sort-by sort-attr items)))))
````


## `reaction`
A `reaction` allows a computation to be dependant on one or several `ratom`s and be updated when change.


## Example
The following creates 3 `ratoms` where the 3rd one will automatically update whenever the first two are updated.

````
(def operand1 (r/atom 4))
(def operand2 (r/atom 5))

(def mult-result (reaction (* @operand1 @operand2)))
````


## `app-db` Example
````
(def app-db
  (reagent/atom  {:items [{:name "a" :val 23 :flag "y"}
                          {:name "b" :val 81 :flag "n"}
                          {:name "c" :val 23 :flag "y"}]
                  :sort-by :name}))
````
