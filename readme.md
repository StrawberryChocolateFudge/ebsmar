# EBSMAR

## Event Based State Management And Rendering

During front end development we often need to work with different rendering and state management solutions like React with Redux. However sometimes more light weight solutions are required in vanilla javascript and we don't want to use frameworks.

This tutorial is a step by step guide on how to write a small and simple framework that combines state management with rendering.

This tutorial explains all the things needed to make it happen. The example in the tutorial is a simple colored box that changes color when clicked on.

First let's create the box.

        <!DOCTYPE html>
        <html lang="en">

        <head>
            <meta charset="UTF-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Document</title>
        </head>
        <style>
            .box {
                width: 100px;
                height: 100px;
                border: 1px solid black;
                margin: 0;
                padding: 0;
                cursor: pointer;
            }
        </style>

        <body>
            <div>
                <div class="box"></div>
            </div>
        </body>

        </html>

The rest of the code goes between the body and the html in the script tag.

## Dispatching and Events

We will use the built in Browser Events. To create a streamlined experience we will dispatch the events on the body.

        //Dispatch an event to trigger something
            function dispatch(to, detail) {
                document.body.dispatchEvent(new CustomEvent(to, { detail }));
            }

We will use different Objects to define what Event we are dispatching and how we route them.
If you use Typescript you should use Enums for this, but for now I'm gonna stick to vanilla JS and use objects.


Define what events we can listen to

        const ListenEvents = { stateChange: "stateChange", render: "render" };

StateChange Events have different types, depending on what variable changes in the state

        const StateChangeType = { setColor: "setColor" };
            
The render events have different types depending on what we render

        const RenderType = { renderBoxColor: "renderBoxColor" }
            
State properties define the state objects properties, used for easy accessing an object based on what property of the state changes!

        const StateProperties = { color: "color" };


We will dispatch events like below. Using the Objects we defined above to help with routing.
We pass a value when setting state to the appropriate funciton. We must create one for each state change.


            // When dispatching state change, it is a value we dispatch
            function dispatch_setColor(value) {
                dispatch(ListenEvents.stateChange, {
                    type: StateChangeType.setColor,
                    value
                });
            }


Rendering functions take props which is a clone of state. We can pass additional variables other than props.             
            
            function dispatch_renderBoxColor(props) {
                dispatch(ListenEvents.render, {
                    type: RenderType.renderBoxColor,
                    props
                })
            }



# The Proxy
Now comes the fun part. We store the state inside a Proxy. The State variable is encapsulated and we can mutate it by dispatching a StateChange Event.


     // Initialize the state in a function. 
        function InitState() {

            // Create a Proxy
            function createState() {
                const state = {
                    color: "white"
                };
                const stateHandler = {
                    set: function (obj, prop, value) {
                        obj[prop] = value;
                        setStateHook[prop](obj, prop, value);
                    }
                }
                return new Proxy(state, stateHandler);
            }
            // The stateContainer is stored inside InitState, to be accessed only by the event listener.

            const stateContainer = createState();

            // The stateSetter helps route the state change event value to the correct place!         
            const stateSetter = {
                [StateChangeType.setColor]: (value) => {
                    stateContainer.color = value;
                }
            }

            document.body.addEventListener(ListenEvents.stateChange, function (event) {
                // We route events based on their type. The StateSetter is used as a switch.
                const type = event.detail.type;
                const value = event.detail.value;
                stateSetter[type](value);
            })
        };


The SetStateHook is an Object we can access with StateProperties

        // The hook inside set state is used for triggering an action on state change, like rendering!
        const setStateHook = {
            [StateProperties.color]: function (obj, prop, value) {
                // Do something on color change...
                // Use Object.assign to copy the state when passing it as props to the renderer 
                dispatch_renderBoxColor(Object.assign({}, obj));
            }
        }

## Rendering

        function InitRenderer() {
                const Render = {
                    [RenderType.renderBoxColor]: async (props) => {
                        // The renderer can use the DOM APIs or even React from here to change the DOM.
                        renderBox(props.color);
                        // The event listeners to DOM elements are also attached during Rendering.
                        boxActions(props)
                    }
                }
                document.body.addEventListener(ListenEvents.render, async (e) => {
                    const type = e.detail.type;
                    const props = e.detail.props;
                    await Render[type](props);
                })
            }


When rendering to DOM we can set event listeners on DOM elements.

        function boxActions(props) {
                // Select the box via className and dispatch a state change
                box.onclick = function () {
                    if (props.color === "red") {
                        dispatch_setColor("blue");
                    } else if (props.color === "blue") {
                        dispatch_setColor("red");
                    }

                }
            }

The rendering function that changes the DOM. I could use React here or Lit-HTML for example.

            function renderBox(color) {
                box.style.backgroundColor = color;
            }



Check out the index.html for the full example and try it in your browser.