D3Kit
======

##Quick Search
* [Installation](https://github.com/rnystrom/D3Kit#installation)
* [Overview](https://github.com/rnystrom/D3Kit#overview)

##Installation

To get started, add D3Kit as a submodule to your project. Make sure to add the submodule from the root of your project.

<code>git submodule add git@github.com:rnystrom/D3Kit.git</code>

Then drag <code>D3Kit.xcodeproj</code> into your project. Location does not matter.

![D3Kit as subproject ](https://github.com/rnystrom/D3Kit/blob/master/images/project.png?raw=true)

Open the **Build Phases** of *your* project. Add D3Kit as a **Target Dependency**.

![Target Dependency ](https://github.com/rnystrom/D3Kit/blob/master/images/dependencies.png?raw=true)

Then add <code>libD3Kit.a</code> to **Link Binary With Libraries**.

![Link Binary with Libraries ](https://github.com/rnystrom/D3Kit/blob/master/images/libraries.png?raw=true)

In *your* project, open the **Build Settings** tab. Search for "User Header Search Paths". Set this value to <code>"${PROJECT_DIR}/D3Kit"</code> (including quotes ). Also set "Always Search User Paths" to YES.

![Search Paths ](https://github.com/rnystrom/D3Kit/blob/master/images/search-paths.png?raw=true)

Find the “Other Linker Flags” option, and add the value -ObjC (no quotes ).

![Flags](https://github.com/rnystrom/D3Kit/blob/master/images/flags.png?raw=true)

In your Prefix.pch file add <code>#import &lt;D3Kit/D3Kit.h&gt;</code> and you're done!

##Overview

The classes that are populated from Blizzard's Diablo 3 API are:

* D3Artisan
* D3Career
* D3Follower
* D3Hero
* D3Item
* D3Rune
* D3Skill

Classes have relationships based on the Blizzard Diablo 3 API. Currently the relations are:

* D3Career <code>has many</code> D3Hero
* D3Career <code>has many</code> D3Artisan
* D3Hero <code>has many</code> D3Item
* D3Hero <code>has many</code> D3Skill
* D3Hero <code>has many</code> D3Follower
* D3Item <code>has one</code> D3Rune

The class <code>D3HTTPClient</code> helps retrieve information from Blizzard's Diablo 3 API is a subclass of [AFNetworking's AFHTTPClient](http://afnetworking.org/Documentation/Classes/AFHTTPClient.html). All functionality of <code>D3HTTPClient</code> is available. However there are helper methods included with each class to retrieve:

* Extra information (ie. D3Career call does not fully populate a D3Hero object by nature of the API )
* Images

To load a career and associated objects pass in an account NSString. This string should be in the format <code>battletag#1234</code> or else <code>failure()</code> will be called. The built-in validation requires a "#" to seperate the account name and number.

``` objective-c
[D3Career getCareerForBattletag:account success:^(D3Career *career ) {
    // ...
} failure:^(NSError *error ) {
    // ...
}];
```

When <code>getCareerForBattletag:success:</code> is called each hero for the career is insantiated but not fully loaded because of the way the Blizzard Diablo 3 API is implemented (which is a good thing, as it keeps the response size small). To finish loading a D3Hero object, call:

``` objective-c
[hero finishLoadingWithSuccess:^(D3Hero *hero ) {
    // ...
} failure:^(NSError *error ) {
    // ...
}];
```

Currently all image requests are instance methods that return a <code>AFImageRequestOperation</code> in the assumption that images will be collected in batches. Ie:

``` objective-c
[hero.items enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop ) {
    UIButton *button = // get corresponding button/imageview
    D3Item *item = (D3Item*)obj;
    AFImageRequestOperation *operation = [correspondingItem requestForItemIconWithImageProcessingBlock:NULL success:^(NSURLRequest *request, NSHTTPURLResponse *response, UIImage *image ) {
        [button setImage:image forState:UIControlStateNormal ];
        // ...
    } failure:^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error ) {
        // ...
    }];
    [mutOperations addObject:operation ];
}];
[[D3HTTPClient sharedClient ] enqueueBatchOfHTTPRequestOperations:mutOperations progressBlock:^(NSUInteger completedOperations, NSUInteger totalOperations ){
    // ...
}completionBlock:^(NSArray *operations ) {
    // ...
}];
```

However it should be trivial to add the <code>AFImageRequestOperation</code> to the queue of <code>D3HTTPClient</code>.

