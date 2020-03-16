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