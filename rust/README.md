## Powerful RUST!

### Android

#### Add NDK Variable

`export NDK_HOME=$ANDROID_HOME/ndk/21.0.6113669`

#### Create NDK Builds

```

mkdir ~/.NDK

$(ANDROID_HOME)/build/tools/make_standalone_toolchain.py --api 26 --arch arm64 --install-dir ~/.NDK/arm64;
$(ANDROID_HOME)/build/tools/make_standalone_toolchain.py --api 26 --arch arm --install-dir ~/.NDK/arm;
$(ANDROID_HOME)/build/tools/make_standalone_toolchain.py --api 26 --arch x86 --install-dir ~/.NDK/x86;

```

#### Config 

Create `cargo-config.toml` and add next lines:

```toml

[target.aarch64-linux-android]
ar = "path_to/NDK/arm64/bin/aarch64-linux-android-ar"
linker = "path_to/NDK/arm64/bin/aarch64-linux-android-clang"

[target.armv7-linux-androideabi]
ar = "path_to/NDK/arm/bin/arm-linux-androideabi-ar"
linker = "path_to/NDK/arm/bin/arm-linux-androideabi-clang"

[target.i686-linux-android]
ar = "path_to/NDK/x86/bin/i686-linux-android-ar"
linker = "path_to/NDK/x86/bin/i686-linux-android-clang"

```

#### Copy cargo config

`cp cargo-config.toml ~/.cargo/config`

#### Add Android Architectures

`rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android`

#### Create Rust lib

`cargo new rust --lib`

#### Build library

```

cargo build --target aarch64-linux-android --release
cargo build --target armv7-linux-androideabi --release
cargo build --target i686-linux-android --release

```

#### Copy libraries

```

# cd android-rust

mkdir app/src/main/jniLibs
mkdir app/src/main/jniLibs/arm64-v8a
mkdir app/src/main/jniLibs/armeabi-v7a
mkdir app/src/main/jniLibs/x86

cp rust/lib/target/aarch64-linux-android/release/librust.so app/src/main/jniLibs/arm64-v8a/librust.so
cp rust/lib/target/armv7-linux-androideabi/release/librust.so app/src/main/jniLibs/armeabi-v7a/librust.so
cp rust/lib/target/i686-linux-android/release/librust.so app/src/main/jniLibs/x86/librust.so

```

#### Or run automatically Android linked script

`cd rust && ./link-android.sh`

#### Add to project

Open up `MainActivity.kt` and paste the following code. 
We declare an external function hello which tells Android to look for a native library function named `Java_com_zikwall_androidrust_MainActivity_hello`. 
Before we can call this function we have to load our library using `System.loadLibrary`.

Example:

```

package com.example.android

import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.util.Log

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        System.loadLibrary("rust")
        Log.d("rust", hello("World"))
    }

    external fun hello(to: String): String
}

```

__Run__ your application in __debug__ mode by clicking on: __Debug app__ and see debug tab console, you looking following line:

`D/rust: Hello World` 

### iOS

#### Getting set up

Before we get started we need to make sure we have the rust toolchain set up. 
We will assume you already have a working iOS toolchain, if not you should download Xcode and set it up according to any other iOS guide. 
To make sure you have the xcode command line tools installed run the following command. 
This all assumes you are running macOS as that is a requirement to build for iOS.

`xcode-select --install`

#### Add iOS Architectures

`rustup target add aarch64-apple-ios armv7-apple-ios armv7s-apple-ios x86_64-apple-ios i386-apple-ios`

#### Cargo dependence

```

cargo install cargo-lipo
cargo install cbindgen

```

#### Project create if not exist & not use previous step

```
cargo new rust --lib
cd rust
```

#### Rust example code

```rust

use std::os::raw::{c_char};
use std::ffi::{CString, CStr};

#[no_mangle]
pub extern fn rust_hello(to: *const c_char) -> *mut c_char {
    let c_str = unsafe { CStr::from_ptr(to) };
    let recipient = match c_str.to_str() {
        Err(_) => "there",
        Ok(string) => string,
    };
    CString::new("Hello ".to_owned() + recipient).unwrap().into_raw()
}

#[no_mangle]
pub extern fn rust_hello_free(s: *mut c_char) {
    unsafe {
        if s.is_null() { return }
        CString::from_raw(s)
    };
}

```

#### Create C bindings

`cbindgen src/lib.rs -l c > rust.h`

#### Edit Cargo.toml

```toml

[lib]
name = "rust"
crate-type = ["staticlib", "cdylib"]

```

#### Xcode

Time to start a new Xcode project and test this out in a simulator. 
Start by going through the standard Xcode project setup, we’ll be using Swift but you can use Objective-C if you want as well. 
We will name the project hello-rust saving it next to our rust library at the root of rust-ios-example. 
Open the generated ViewController.swift and replace its contents with the following.


```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let result = rust_hello("world")
        let swift_result = String(cString: result!)
        rust_hello_free(UnsafeMutablePointer(mutating: result))
        print(swift_result)
    }
}

```

#### Copy

```

cd ~/rust-ios-example # or wherever you created this project

mkdir hello-rust/libs
mkdir hello-rust/include

cp rust/rust.h hello-rust/include
cp rust/target/universal/release/librust.a hello-rust/libs

```

#### Or run automatically iOS linked script

`cd rust && ./link-ios.sh`