[https://github.com/yue19870813/ituuz-x](https://github.com/yue19870813/ituuz-x)

# lightMVC

lightMVC will be integrated into this project in the future. Please follow this project.

---

Framework for Cocos Creator

A simple lightweight MVC framework suitable for small to medium-sized projects. The subsequent lightMVC_ex will be expanded to adapt to the development of large projects. This lightweight MVC framework can help developers organize code and business structures, making projects easier to maintain and extend, and improving development efficiency. There are complete example demos in the examples directory.

#### Architecture Diagram
![Architecture Diagram](./mvc.png)

#### Node Functions
- Facade: Global control class, holding management objects for MVC layers. In principle, except for initializing the framework by calling `init` and running the first scene, no interface or property in Facade should be referenced or called. This class is a global singleton object, containing several important interfaces as follows:

```javascript
/**
 * Initialize framework configuration
 * @param {boolean} debug Whether it is in debug mode
 * @param {cc.Size} designResolution Design resolution
 * @param {boolean} fitHeight Whether to fit the height
 * @param {boolean} fitWidth Whether to fit the width
 */
public init(debug: boolean, designResolution: cc.Size, fitHeight: boolean, fitWidth: boolean): void;

/**
 * Run scene
 * @param {{new(): BaseMediator}} mediator Scene mediator type, class type.
 * @param {{new(): BaseScene}} view Scene view type, class type.
 * @param {Object} data Custom arbitrary type pass-through data. (Optional)
 * @param {()=>void} cb Callback function when loading is complete.
 */
public runScene(mediator: {new(): BaseMediator}, view: {new(): BaseScene}, data?: any, cb?: ()=>void): void;
```

- Model: Data object, used to process data logic and store data, often used to interact with the server data, and refresh the display with the View layer through messaging. The main interfaces are as follows:

```javascript
/** Interface called when Model is initialized, can be used to initialize some data */
public init(): void;

/**
 * Send message interface, should be called when data changes to send messages to refresh the View layer.
 * @param {string} noti Message name
 * @param {Object} data Message data
 */
public sendNoti(noti: string, data?: any): void;

/** Cleaning interface, subclasses can implement cleaning logic */
public clear(): void;
```

- View: Display layer, displays according to business logic and data, handles user input, and interacts with other layers through events. The main interfaces are as follows:

```javascript
/** Called when View is created, subclasses can override */
public init(): void;

/**
 * Send UI event, logic layer receives event processing logic.
 * @param {string} event Event name
 * @param {Object} body Event parameters
 */
public sendEvent(event: string, body?: any): void;

/** Close the current interface */
public closeView(): void;

/** Close all popped up interfaces */
public closeAllPopView(): void;

/** Called when the interface is closed, subclasses can override this method */
public onClose(): void;

/** Subclasses override, returning the prefab path of the UI, default is an empty node */
public static path(): string;
```

- Mediator: Logical layer mediator, responsible for receiving notifications from the Model layer to refresh the View layer display, and also for receiving events from the View layer to handle user input, and processing data layer data through Commands. The main interfaces are as follows:

```javascript
/**
 * Initialization interface, at this time the view has not been created yet, if you want to operate the view, please do it in the viewDidAppear function.
 * @param {Object} data Custom arbitrary type pass-through data. (Optional)
 * @override
 */
public init(data?: any): void;

/**
 * Interface called after the view is displayed
 * @override
 */
public viewDidAppear(): void;

/**
 * Bind UI events, receive events dispatched by the view layer
 * @param {string} name Event name
 * @param {(any)=>void} cb Event callback
 * @param {BaseMediator} target Callback binding object
 */
public bindEvent(name: string, cb: (body: any)=>void, target: BaseMediator): void;

/**
 * Register notification listener
 * @param {string} noti Notification key value
 * @param {(data: any)=>void} cb Notification listening callback function
 * @param {Object} target Object bound to the callback
 */
public registerNoti(noti: string, cb: (data: any)=>void, target: any): void;

/**
 * Send notification message
 * @param {string} noti Notification key value
 * @param {Object} body Message parameter
 */
public sendNoti(noti: string, body: any): void;

/**
 * Send command interface
 * @param {{new (): BaseCommand}} cmd Command class
 * @param {Object} data Command parameter
 */
public sendCmd<T extends BaseCommand>(cmd: {new (): T}, data?: any): void;

/**
 * Open a new scene
 * @param data {Object} Custom arbitrary type pass-through data. (Optional)
 */
public runScene(mediator: {new(): BaseMediator}, view: {new(): BaseScene}, data?: any): void;

/**
 * Return to the previous scene
 * @returns {boolean} Whether the previous scene exists
 */
public backScene(): boolean;

/**
 * Open view interface
 * @param {{new(): BaseMediator}} mediator Interface mediator type, class type.
 * @param {{new(): BaseView}} view View scene mediator type, class type.
 * @param {Object} data Custom arbitrary type pass-through data. (Optional)
 */
public popView(mediator: {new(): BaseMediator}, view: {new(): BaseView}, data?: any): void;

/**
 * Add hierarchy
 * @param {{new(): BaseMediator}} mediator Interface mediator type, class type.
 * @param {{new(): BaseView}} view View scene mediator type, class type.
 * @param {number} zOrder Layer level. (Optional)
 * @param {Object} data Custom arbitrary type pass-through data. (Optional)
 */
public addLayer(mediator: {new(): BaseMediator}, view: {new(): BaseView}, zOrder?: number, data?: any): void;

/** Get model object */
public getModel<T extends BaseModel>(model: {new (): T}): T;

/** Destruction interface */
public destroy(): void;
```

#### Usage
1. Initialize the framework:

```javascript
// Debug mode is false, design resolution is 1080*2048, width adaptation.
Facade.getInstance().init(false, cc.size(1080, 2048), false, true);
```

2. Register model data objects:

```javascript
// If data layer is needed, then all the models needed should be registered first.
Facade.getInstance().registerModel(PlayerModel);
```

3. Run the first scene:

```javascript
// When running the first scene, call the runScene interface of Facade, pass in the Mediator and Scene to be run, and optionally pass in parameters.
Facade.getInstance().runScene(DefaultSceneMediator, DefaultScene, "Test Parameter 999");
```

4. In principle, except for the above three steps that need to reference Facade, Facade does not need to be called afterwards. Handle corresponding logic in different MVC layers, and all parent class interfaces are supported.

5. After the scene is running, you can create layer views or pop up views in the scene Mediator. The difference between Layer view and pop view is that they are managed by two managers. We consider Layer as a view interface that is initialized and created within the scene and will not be closed, while pop view is a view interface that can be opened or closed at any time. You can handle it flexibly. For example, in DefaultSceneMediator:

```javascript
/**
 * Create a permanent view interface FirstView
 * this.addLayer is the basic function interface provided by BaseMediator (more interfaces can be viewed in the source code).
 * The hierarchy is 1, and the parameter this._data is passed in.
 * */
this.addLayer(FirstMediator, FirstView, 1, this._data);
```

6. UI node operation interface in View layer. In View, there is a member property `ui`, and the UI nodes of this interface will be automatically initialized to this member property during initialization. When operating UI nodes, you can use this property for operation. The property type is `UIContainer`, and the commonly used interfaces are `getNode` and `getComponent`. Example code:

```javascript
// Get node
let closeBtnNode = this.ui.getNode("close_btn");
closeBtnNode.on(cc.Node.EventType.TOUCH_END, this.closeAllView, this);
// Get Component
let desLabel = this.ui.getComponent("des_label", cc.Label);
desLabel.string = "test";
```

7. Event interaction between View layer and Mediator layer. Mediator directly holds the reference of View, so you can directly call the interface in View. However, View and Mediator need to interact through events (Event). First, register the listener in Mediator:

```javascript
this.bindEvent(FirstView.OPEN_B, (str: string)=>{
    // todo something...
}, this);
```

Then send events to notify Mediator in View through `sendEvent` interface:

```javascript
// The first parameter is the event name, and the second parameter is the parameter passed.
this.sendEvent(FirstView.OPEN_B, "BBB");
```

8. Mediator operates Model data. In Mediator, you can use the `getModel` interface to get the specified Model object, and directly reference to read the data in Model. There are two ways to modify data. One is to directly modify through the reference of Model, which is mostly used for simple direct modification of some values; the other is more complex, such as getting data from multiple Models for complex logical operations and modifying multiple values, which is suitable for encapsulating logic into a Command (Command), and processing data through sending commands, which can reduce the complexity and coupling of logic in Mediator. Example:

```javascript
// Direct modification through reference
let playerModel = this.getModel(PlayerModel);
this.view.setLevelDisplay(playerModel.getPlayerLv());

// Operation through command
this.sendCmd(UpdateExpCommand, exp);
```

9. Model data modification notifies View to refresh logic. In most cases, Model is used to process pure data logic and data interaction with the server. When data changes, we hope to notify View to refresh the display. In this case, we can only notify Mediator through throwing message notification, and then modify View display through Mediator. First, register message notification in Mediator:

```javascript
this.registerNoti(Notification.UPDATE_EXP_FINISH, ()=>{
    // todo something ...
}, this);
```

Then notify Mediator in Model through sending this message notification:

```javascript
// The second parameter of this interface can pass parameters
this.sendNoti(Notification.UPDATE_EXP_FINISH);
```

10. Interaction between Mediators is very simple. Just use the way Model sends notifications to Mediators mentioned above.

#### Others
Simple interaction rules and interface calls are provided, and the organization of code structure is also very important, which depends on the reasonable arrangement of each person or project, after all, it is a matter of each person's opinion. **There are complete example demos in the examples directory**.

lightMVC is currently only suitable for small to medium-sized projects. Coping with overly complex large projects may be a bit challenging, but it will continue to be maintained and expanded to [lightMVC_ex](./lightMVC_ex/README.md) to support large project development, and lightMVC will always remain simple and lightweight.

Feel free to provide feedback on any issues or improvements in the framework.
