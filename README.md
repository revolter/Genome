
<p align="center">
  <img src="/Images/GenomeBanner.png" width=800></img>
</p>

<h2 align="center">Failure-Driven Object Mapping in Swift</h2>

Genome has gone Swift!  If you're looking for the original, ObjC implementation, you can find it <a href="https://github.com/LoganWright/Genome/tree/0.1">here</a>!  The ObjC version is no longer maintained, and any new developments will be done in Swift.

<h1 align="center">Genome 2.0.0</h1>

With the 2.0.0 release, there are some breaking syntax adjustments that you should be aware of.  One of the major changes is the removal of the dreaded and confusing `<~?` operator.  By structuring differently, special cases for optionality are no longer necessary, and all direct set mappings can use `<~` or the newly added `extract` function.  This also positions the library to better adapt when containers of conforming objects can conform themselves. The goal here would be reducing the amount of overload functions necessary.

<h1 align="center">Genome 2.1.0</h1>

With the 2.1.0 release, `Json` has been replaced with a file type independent `Node` object. This positions the library to gain serializers and deserializers for various file types other than JSON. NSData support has been temporarily removed, as the dependency that provided it has been phased out. It will be re-added with the serialization and deserialization support. `NSJSONSerialization` can be used perform the conversion from `NSData` to `[String:AnyObject]` while serialization support is coming.

#### Pure

Removing Foundation dependencies for core functionality has always been a goal of this library, and it turns out that it snuck into the `1.0.0` version.  By casting `AnyObject` to and from value types such as `String`, `Int`, etc. we were dependent on the underlying `NSString`, `NSNumber`, `NSArray`, etc. class systems.  

This means that going forward, we'll be using the new `Node` type.  The new `Node` type is an independent structure, and no longer a `typealias` of `[String : AnyObject]`.  Usage should be natural with comprehensive literal syntax: `let name: Node = "HumanName"`.  When converting from data or a string, use `let node = try Node.deserialize(nodeData)`.  

There are periodic changes and maintenance throughout, so the README is definitely worth a skim to see some of what's new. Remember, if you're feeling nostalgic and you're not ready to update, you can roll back to a 1.0.0 compatible version by using `pod 'CocoaPods', '~> 1.0.0'`.

If you're not using CocoaPods, check the <a href="https://github.com/LoganWright/Genome/releases">releases</a> section and find a `1.0.0` compatible version.

Happy Mapping!

### Building Project

Just build the project using the Genome.xcodeproj file in the root of the repository.

### Why

With great libraries like <a href="https://github.com/thoughtbot/Argo">Argo</a> and <a href="https://github.com/Hearst-DD/ObjectMapper">ObjectMapper</a>, why do we need another? Ultimately, I wanted to build it, and I wanted something a little different.  

The goal of this library is to satisfy the following constraints:

>\- Customizable Initialization

>\- Flexible Error Handling

>\- Failure Driven

>\- Automatic Nested Mapping

>\- Simple To Use

>\- Two-Way Serialization

>\- Transformable Values

>\- Type Safety

>\- Constants (`let`)

>\- Independent of Foundation Framework (Supports Linux)

>\- Struct Friendly

>\- Inheritance Friendly

>\- Core Data and Persistence Compatible

### Playground / Examples

The <a href="/Genome.playground">playground</a> provided by this project can be used to test the library.  It also provides some examples on how to use the library.  

### Failure Driven

With the introduction of Swift 2.0, we were given an entirely new error handling system, and a new keyword `try`.  In mapping data to models, there are, unfortunately, many points of failure.  By being very explicit about the failability of these operations, we can be confident that our code will run as expected, and gain clarity into error messages earlier in the process.  This means that we're going to have to write the word `try` quite a bit in the name of safety.

### Initial Setup

If you wish to install the library manually, you'll need to find the source files located in the playground's <a href="/Genome.playground/Sources">sources</a>  directory.

It is highly recommended that you install Genome through <a href="https://www.cocoapods.org">CocoaPods.</a>  Here is a personal CocoaPods reference just in case it may be of use: <a href="https://gist.github.com/LoganWright/5aa9b3deb71e9de628ba">CocoaPods Setup Guide</a>

```Ruby
pod 'Genome', '~> 2.0.0'
```

You can also install Genome using [Carthage](https://github.com/Carthage/Carthage). Just add the line below to your `Cartfile`:

```
github "LoganWright/Genome"
```

And execute `carthage update` to download and compile the framework.

### Table Of Contents

* [Quick Start](#quick-start)
* [Node](#node)
* [Inheritance](#inheritance)
* [NodeConvertibleType](#nodeconvertibletype)
* [Instantiation](#instantiation)
* [Alamofire](#alamofire)
* [Core Data](#core-data)
* [Logging](#logging)

### Quick Start

Let's take the following hypothetical JSON

```Swift
[
    "name" : "Rover",
    "nickname" : "RoRo", // Optional Value
    "type" : "dog"
]
```

Here's how we might create the model for this


```Swift
enum PetType : String {
    case Dog = "dog"
    case Cat = "cat"
}

struct Pet : MappableObject {
    let name: String
    let type: PetType
    let nickname: String

    init(map: Map) throws {
        name = try map.extract("name")
        nickname = try map.extract("nickname")
        type = try map["type"]
            .fromNode { PetType(rawValue: $0)! }
    }

    func sequence(map: Map) throws {
        try name ~> map["name"]
        try type ~> map["type"]
            .transformToNode { $0.rawValue }
        try nickname ~> map["nickname"]
    }
}
```

### `Map`

This is the object that is used two encapsulate the `Node` as well as a more global context which can represent any object you may want your sub operations to have access to.

This has particular use in things like CoreData where a `NSManagedObjectContext` may be required.

### `MappableObject`

This is one of the core protocol options for this library.  It will be the go to for most standard mapping operations.

It has two requirements

#### `init(map: Map) throws`

This is the initializer you will use to map your object.  You may call this manually if you like, but if you use any of the built in convenience initializers, this will be called automatically.  Otherwise, if you need to initialize a `Map`, use:

```Swift
let map = Map(dan: someNode, context: someContext)
```

It has two main requirements

#### `sequence(map: Map) throws`

The `sequence` function is called in two main situations. It is marked `mutating` because it will modify values on `fromNode` operations.  If however, you're only using sequence for `toNode`, nothing will be mutated and one can remove the `mutating` keyword. (as in the above example)

`Note, if you're only mapping to Node, nothing will be mutated.`

##### FromNode

When mapping to Node w/ any of the convenience initializer.  After instantiating the object, `sequence` will be called.  This allows objects that don't initialize constants or objects that use the two-way operator to complete their mapping.

> If you are initializing w/ `init(map: Map)` directly, you will be responsible for calling `sequence` manually if your object requires it.

It is marked `mutating` because it will modify values.

`Note, if you're only mapping to Node, nothing will be mutated.`

##### ToNode

When accessing an objects `nodeRepresentation()`, the sequence operation will be called to collect the values into a `Node` package.

### `~>`

This is one of the main operations used in this library.  The `~` symbolizes a connection, and the `<` and `>` respectively symbol a flow of value.  When declared as `~>` it symbolizes that mapping only happens from value, to Node.

You could also use the following:

| Operator | Directions | Example | Mutates |
|:---:|:---:|:---:|:---:|
|`<~>`| To and From Node | `try name <~> map["name"]` | ✓ |
|`~>`| To Node Only | `try clientId ~> map["client_id"]` | 𝘅 |
|`<~`| From Node Only | `try updatedAt <~ map["updated_at"]` | ✓ |

### `transform`

Genome provides various options for transforming values.  These are type-safe and will be checked by the compiler.

These are chainable, like the following:

```Swift
try type <~> map["type"]
    .transformFromNode {
        return PetType(rawValue: $0)
    }
    .transformToNode {
        return $0.rawValue
    }
```

>Note: At the moment, transforms require absolute optionality conformance in some situations. ie, Optionals get Optionals, ImplicitlyUnwrappedOptionals get ImplicitlyUnwrappedOptionals, etc.

#### `fromNode`

When using `let` constants, you will need to call a transformer that sets the value instantly.  In this case, you will call `fromNode` and pass any closure that takes a `NodeConvertibleType` (a standard Node type) and returns a value.

#### `transformFromNode`

Use this if you need to transform the node input to accomodate your type.  In our example above, we need to convert the raw node to our associated enum.  This can also be appended to mappings for the `<~` operator.

#### `transformToNode`

Use this if you need to transform the given value to something more suitable for data.  This can also be appended to mappings for the `~>` operator.

### `try`

Why is the `try` keyword on every line!  Every mapping operation is failable if not properly specified.  It's better to deal with these possibilities, head first.  

For example, if the property being set is non-optional, and `nil` is found in the `Node`, the operation should throw an error that can be easily caught.

# More Concepts

Some of the different functionality available in Genome

The way that Genome is constructed, you should never have to deal w/ `Node` beyond deserializing and serializing for your web services.  It can still be used directly if desired.

## Inheritance

Genome is most suited to `final` classes and structures, but it does support Inheritance.  Unfortunately, due to some limitations surrounding generics, protocols, and `Self` it requires some extra effort.

### `Object`

The `Object` type is provided by the library to satisfy most inheritance based mapping operations.  Simply subclass `Object` and you're good to go:

```Swift
class MyClass : Object {}
```

> Note: If you're using `Realm`, or another library that has also used `Object`, don't forget that these are module namespaced in Swift.  If that's the case, you should declare your class: `class MyClass : Genome.Object {}`

### `Custom`

If you're using a custom class, you'll need to add some additional functions.  Here's what a basic base class might look like:

```Swift
class CustomBase : MappableBase {
    required init() {}

    static func newInstance(node: Node, context: Context) throws -> Self {
        let map = Map(node: node, context: context)
        let new = self.init()
        try new.sequence(map)
        return new
    }

    func sequence(map: Map) throws {}
}
```

**Notice the `required` initializer above.  When returning `Self` at a class level, you will almost always need a required initializer.**

If you need to extend an existing base class, and for particularly complex situations, see the CoreData example below as reference.

### `BasicMappable`

In order to support flexible customization, Genome provides various mapping options for protocols.  Your object can conform to any of the following.  Although each of these initializers is marked with `throws`, it is not necessary for your initializer to `throw` if it is guaranteed to succeed.  In that case, you can omit the `throws` keyword safely.

| Protocol | Required Initializer |
|:---|:---|
| BasicMappable | `init() throws` |
| MappableObject | `init(map: Map) throws` |

These are all just convenience protocols, and ultimately all derive from `MappableBase`.  If you wish to define your own implementation, the rest of the library's functionality will still apply.

### `NodeConvertibleType`

This is the true root of the library.  Even `MappableBase` mentioned above inherits from this core type.  It has two requirements:

```Swift
public protocol NodeConvertibleType {
    static func newInstance(node: Node, context: Context) throws -> Self
    func nodeRepresentation() throws -> Node
}
```

All Node basic types such as `Int`, `String`, etc. conform to this protocol which allows ultimate flexibility in defining the library.  It also paves the way to much fewer overloads going forward when collections of `NodeConvertibleType` can also conform to it.

> This can be used as a supplement to `transform` types mentioned above.  If an object conforms to this protocol, it will be immediately useable within the library.

## Instantiation

If you are using the standard instantiation scheme established in the library, you will likely initialize with this function.

```Swift
public init(node: Node, context: Context = EmptyNode) throws
```

Now we can easily create an object safely:

```Swift
do {
    let rover = try Pet(node: node_rover)
    print(rover)
} catch {
    print(error)
}
```

If all we care about is whether or not we were able to create an object, we can also do the following:

```Swift
let rover = try? Pet(node: node_rover)
print(rover) // Rover is type: `Pet?`
```

### Context

`Context` is defined as an empty protocol that any object you might need access to can conform to and passed within.

### Foundation

If you're using `Foundation`, you can also use the following initialization:

```Swift
public init(node: AnyObject, context: [String : AnyObject] = [:]) throws

public init(node: [String : AnyObject], context: [String : AnyObject] = [:]) throws
```

### CollectionTypes

You can instantiate collections directly w/o mapping as well:

```Swift
let people = try [People](node: someNode)
```

### Class Level Instantiation

See Core Data

### `mappedInstance(node: Node)`

This is the function that should be used to initialize new mapped objects for a given node.

### Playground

Feel free to check out and interact with the <a href="/GenomePlayground.playground">playground</a> provided in this repo!

### Alamofire

Here's a quick example of using Genome alongside <a href="https://github.com/Alamofire/Alamofire">Alamofire</a> (3.0)

```Swift
import Alamofire
import Genome

struct NasaPhoto : BasicMappable {
    private(set) var title: String = ""
    private(set) var mediaType: String = ""
    private(set) var explanation: String = ""
    private(set) var concepts: [String] = []

    private(set) var imageUrl: NSURL!

    mutating func sequence(map: Map) throws {
        try title <~ map["title"]
        try mediaType <~ map ["media_type"]
        try explanation <~ map["explanation"]
        try concepts <~ map["concepts"]
        try imageUrl <~ map["url"]
            .transformFromNode {
                return NSURL(string: $0)
            }
    }
}

enum NasaResult<T> {
    case Success(T)
    case Failure(ErrorType)
}

struct Nasa {
    static func fetchPictureOfTheDay(completion: NasaResult<NasaPhoto> -> Void) {
        let url = "https://api.nasa.gov/planetary/apod?concept_tags=True&api_key=DEMO_KEY"
        Alamofire.request(.GET, url)
            .responseJSON { response in
                switch response.result {
                case .Success(let value):
                    do {
                        let photo = try NasaPhoto(node: value)
                        completion(.Success(photo))
                    } catch {
                        completion(.Failure(error))
                    }
                case .Failure(let error):
                    completion(.Failure(error))
                }
        }
    }
}
```

Now, when we want to use our `NasaPhoto` object, we can use it knowing that it will be safe.

```Swift
Nasa.fetchPictureOfTheDay { [weak self] result in
    switch result {
    case .Success(let photo):
        self?.navigationItem.title = photo.title
        self?.descriptionLabel.text = photo.explanation
        self?.imageView.sd_setImageWithURL(photo.imageUrl)
    case .Failure(let error):
        print("Error: \(error)")
    }
}
```

#### Core Data

If you wish to use `CoreData`, you will want to add something similar to the following to your project:

```Swift

import CoreData

extension NSManagedObjectContext : Context {}

public class NSMappableManagedObject: NSManagedObject, MappableBase {
    public class var entityName: String {
        return "\(self)"
    }

    public func sequence(map: Map) throws {
        fatalError("Sequence must be overwritten")
    }

    public class func newInstance(node: Node, context: Context) throws -> Self {
        return try newInstance(node, context: context, type: self)
    }

    public class func newInstance<T: NSMappableManagedObject>(node: Node, context: Context, type: T.Type) throws -> T {
        let context = context as! NSManagedObjectContext
        let new = NSEntityDescription.insertNewObjectForEntityForName(entityName, inManagedObjectContext: context) as! T
        let map = Map(node: Node, context: context)
        try new.sequence(map)
        return new
    }
}
```

The generics above might seem a little strange, but they are an attempt to work within the extremely strict inheritance / `Self` system established by Swift without using a `required` initializer.  This type of format can be used for other persistence layers that have class level initializers or object creation factories.

#### Custom Implementation

Feel free to implement your own protocol by inheriting from `Mappable` and defining the initialization scheme.  Look at the implementations of the provided protocols for information on how to do this.

### Logging

All errors are passed through a logging system before being thrown.  This allows for helpful debugging and allows the potential to add remote logging to your project.

#### Adding Loggers

You can add any logger by conforming to `ErrorType -> Void`.  Here's a quick example of how we might implement this:

```Swift
func reportErrorToServer(error: ErrorType) {
    // ... handle the error here
}
```

Then, add it to the loggers:

```Swift
loggers.append(reportErrorToServer)
```

If you're using `Genome` through modules, it can be more clear to acknowledge the namespace:


```Swift
Genome.loggers.append(reportErrorToServer)
```

#### Turning Loggers Off

Just set `loggers` to an empty array and the system will no longer print to the console.

```Swift
loggers = []

// or namespaced

Genome.loggers = []
```
