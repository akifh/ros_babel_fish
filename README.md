# ros_babel_fish

This library enables ROS nodes written in C++ to communicate using message types that are unknown at compile time.

You can subscribe and even publish any available message type.  
It also supports both providing and calling a service.

You most likely don't need this library.  
Possible use cases where you do need it are:
* UIs displaying the content of various at compile time unknown messages
* Plugins for script languages that can not access the C++ message definitions without modification
  * Spot for shameless self-advertising: Check out my ROS QML plugin which uses this library to allow subscribing, publishing and more directly in QML 

The main focus of this library was usability but it is also very performant since it uses a lazy copy mechanism for bigger fields such as big arrays.  
Instead of copying the message it will retain a pointer at the start of the field. A copy is only made if explicitly requested or a value in the
field is changed.  
Check out the examples in the example folder to find out how to use this library.

Here's an example on how to publish a message:

```
NodeHandle nh;
BabelFish fish;
// Advertise a publisher on topic /pose
ros::Publisher pub_pose = fish.advertise( nh, "geometry_msgs/Pose", "/pose", 1, true );

Message::Ptr message = fish.createMessage( "geometry_msgs/Pose" );
auto &compound = message->as<CompoundMessage>();
compound["position"].as<CompoundMessage>()["x"].as<ValueMessage<double>>().setValue(1.1);
// or using convenience methods
compound["position"]["y"].as<ValueMessage<double>>().setValue(2.4);
// or using even more convenience methods
compound["position"]["z"] = 3.6;
// Be careful with your types here. Casting to a wrong type will throw an exception!
// The as<ValueMessage<T>> method is also a bit faster because the convenience method
// will automatically convert to the right type and perform bound and compatibility checks.
// This makes it more robust but comes with a little overhead.
// Note that assigning a double to a float field will always throw an exception because
// the float may not have the required resolution.
// Assigning, e.g., an int to a uint8 field will only throw if the int is out of bounds (0-255)
// otherwise a warning will be printed because uint8 is not compatible with all possible values
// of int. This warning can be disabled as a compile option. 

compound["orientation"]["w"] = 0.384;
compound["orientation"]["x"] = -0.003;
compound["orientation"]["y"] = -0.876;
compound["orientation"]["z"] = 0.292;

BabelFishMessage::Ptr translated_message = fish.translateMessage( message );
pub_pose.publish( translated_message );
```

## Current TODOs

- [x] Subscribing topics with at compile time unknown message type
- [x] Publishing messages with at compile time unknown message type
- [x] Calling a service with at compile time unknown service definition
- [x] Advertising a service with at compile time unknown service definition
- [x] Creating a service client with at compile time unknown service definition
- [x] Add more error checks to fail gracefully in case of errors
- [ ] Documentation
- [ ] Calling actions
- [ ] Providing an action server(?)
