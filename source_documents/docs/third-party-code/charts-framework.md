
## Charts libraries installation

### How Are They Integrated?

Charts, swift-algorithms and swift-numerics are integrated in the US Walmart target and Health Trackers plugin. 

- **Charts.xcframework**
    It is embed and sign in the Walmart US target and Health Trackers plugin, it is the core of the library to display Charts with different shapes.
- **Algortithms.xcframework**
    It is embed and sign in the Walmart US target and Health Trackers plugin, it is required for the correct functioning of  the Charts library.
- **NumericsShims.xcframework**
    It is embed and sign in the Walmart US target and Health Trackers plugin, it is required for the correct functioning of the Charts library.
- **RealModule.xcframework**
    It is embed and sign in the Walmart US target and Health Trackers plugin, it is required for the correct functioning of the Charts library.

### Generate XCFrameworks

Prerequisites

* Carthage version 0.38
* Xcode 14.1


#### Charts version 4.1.0

1. Clone the repository https://github.com/danielgindi/Charts and checkout to the 4.1.0 version

    ```sh 
        git clone https://github.com/danielgindi/Charts.git
        git checkout v4.1.0
    ```
    
    
2. To generate the XCFramework for iOS platform we need to execute the following scripts

    ```sh 
        carthage build --no-skip-current --platform iOS --use-xcframeworks --no-use-binaries

        carthage archive Charts
    ```

3. This script will generate a Charts.xcframework.zip file that we will use later

4. We have to remove the dSYMs for the generated framework on both folders ios-arm64 and ios-arm64_x86_64-simulator this
   will help us to avoid problems with absolute paths [Reference](https://steipete.com/posts/couldnt-irgen-expression/)



####  Swift-algorithms version 1.0.0

1.    Clone the repository https://github.com/apple/swift-algorithms

   ```sh
       git clone https://github.com/apple/swift-algorithms.git
   ```     
    

2.   To generate the XCFramework for iOS platform we need to execute the following scripts
     ```sh 
        carthage build --no-skip-current --platform iOS --use-xcframeworks --no-use-binaries

        carthage archive Algorithms
     ```

3. This script will generate an Algorithms.xcframework file that we need to compress it to generate an  Algorithms.xcframework.zip that we will use later

4. We have to remove the dSYMs for the generated framework on both folders ios-arm64 and ios-arm64_x86_64-simulator this
   will help us to avoid problems with absolute paths [Reference](https://steipete.com/posts/couldnt-irgen-expression/)


####  Swift-numerics version 1.0.2

1. Clone the repository https://github.com/apple/swift-numerics
    ```sh 
        git clone https://github.com/apple/swift-numerics.git
    ```
2. Create a xcodeproj file using the following command and build every target manually
    ```sh 
        swift package generate-xcodeproj
    ```

3. This library is composed by various libraries like Numerics, RealModule, NumericsShims, IntegerUtilities and ComplexModule so we need to create an XCFramework for each one.

In order to build the libraries for the iOS platform and the iOS simulator we need to run the following scripts:

    
      xcodebuild archive -scheme swift-numerics-Package -sdk iphoneos -destination 'generic/platform=iOS' -archivePath ./build/iphoneos.xcarchive SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES

      xcodebuild archive -scheme swift-numerics-Package -sdk iphonesimulator -archivePath ./build/iphonesimulator.xcarchive SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES
    
    
4. After running this scripts 2 new files are generated iphoneos.xcarchive and iphonesimulator.xcarchive inside of the build folder and we need to execute 3 scripts to generate an XCFramework for RealModule, NumericsShims and Numerics 

    Real Module

    ```sh 
    xcodebuild -create-xcframework \
    -framework ./build/iphoneos.xcarchive/Products/Library/Frameworks/RealModule.framework \
    -framework ./build/iphonesimulator.xcarchive/Products/Library/Frameworks/RealModule.framework \
    -output ./build/RealModule.xcframework
    ```

    Numerics Shims

    ```sh 
    xcodebuild -create-xcframework \
    -framework ./build/iphoneos.xcarchive/Products/Library/Frameworks/_NumericsShims.framework \
    -framework ./build/iphonesimulator.xcarchive/Products/Library/Frameworks/_NumericsShims.framework \
    -output ./build/NumericsShims.xcframework
    ```

    Numerics 

    ```sh 
    xcodebuild -create-xcframework \
    -framework ./build/iphoneos.xcarchive/Products/Library/Frameworks/Numerics.framework \
    -framework ./build/iphonesimulator.xcarchive/Products/Library/Frameworks/Numerics.framework \
    -output ./build/Numerics.xcframework
    ```


5. This will generate 3 files RealModule.xcframework, NumericsShims.xcframework and Numerics.xcframework that we need to compress to generate the RealModule.xcframework.zip, NumericsShims.xcframework.zip and Numerics.xcframework.zip that we will use later



### Deploy on static-artifact-deployer


In order to integrate third party libraries we need to deploy it on the static-artifact-deployer 

We have to fork the repository and submit a pull request to make changes, once we generated the zip files with the XCFrameworks on it, we have to create folders with the files inside and modify the looper.yml file with the reference to the zip files

    Charts(ui Deploy Charts):

    - (name Deploy Framework) ./deploy-framework.sh Charts Charts 4.1.0 repository/com/dcg/Charts/4.1.0/Charts.xcframework.zip


    SwiftAlgorithms(ui Deploy swift-algorithms):

    - (name Deploy Framework) ./deploy-framework.sh apple.swift-algorithms Algorithms 1.0.0 repository/apple/swift-algorithms/Algorithms/1.0.0/Algorithms.xcframework.zip


    SwiftNumerics(ui Deploy swift-numerics):

    - (name Deploy Framework) ./deploy-framework.sh apple.swift-numerics Numerics 1.0.2 repository/apple/swift-numerics/Numerics/1.0.2/Numerics.xcframework.zip    


    SwiftNumericsShims(ui Deploy swift-numerics-shims):

    - (name Deploy Framework) ./deploy-framework.sh apple.swift-numerics NumericsShims 1.0.2 repository/apple/swift-numerics/NumericsShims/1.0.2/NumericsShims.xcframework.zip


     SwiftNumericsRealModule(ui Deploy swift-numerics-real-module):

    - (name Deploy Framework) ./deploy-framework.sh apple.swift-numerics RealModule 1.0.2 repository/apple/swift-numerics/RealModule/1.0.2/RealModule.xcframework.zip




We will open https://ci.walmart.com/job/ios-walmart-deploy-static-framework/  this URL and If everything is correct we will see our frameworks ready to deploy on proximity, we will see the following options:

* Deploy swift-algorithms
* Deploy swift-numerics
* Deploy swift-numerics-shims
* Deploy swift-numerics-real-module
* Deploy Charts

If the deploy is correct we will see the generated URL’s 

* https://repository.walmart.com/content/repositories/pangaea_releases/Charts/Charts/4.1.0/Charts-4.1.0.zip

* https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/Numerics/1.0.2/Numerics-1.0.2.zip

* https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/NumericsShims/1.0.2/NumericsShims-1.0.2.zip

* https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/RealModule/1.0.2/RealModule-1.0.2.zip


On the glass project, we will generate the json files on the BinaryDependencySpecification folder with reference to the URL’s

    Charts.json

    {
    "4.1.0": "https://repository.walmart.com/content/repositories/pangaea_releases/Charts/Charts/4.1.0/Charts-4.1.0.zip"    
    }


    Algorithms.json

    {
    "1.0.0": "https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-algorithms/Algorithms/1.0.0/Algorithms-1.0.0.zip"    
    }

    Numerics.json

    {
    "1.0.2": "https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/Numerics/1.0.2/Numerics-1.0.2.zip"    
    }

    NumericsShims.json

    {
    "1.0.2": "https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/NumericsShims/1.0.2/NumericsShims-1.0.2.zip"    
    }


    RealModule.json
    
    {
    "1.0.2": "https://repository.walmart.com/content/repositories/pangaea_releases/apple/swift-numerics/RealModule/1.0.2/RealModule-1.0.2.zip"    
    }

And on the Cartfile file we will update the references to our files

* binary “BinaryDependencySpecification/Algorithms.json"
* binary “BinaryDependencySpecification/Charts.json"
* binary "BinaryDependencySpecification/Numerics.json"
* binary “BinaryDependencySpecification/NumericsShims.json"
* binary “BinaryDependencySpecification/RealModule.json"

After that we will run the following scripts for the glass project to update the libraries 

    
    glass environment destroy
    
    glass environment setup 
    

If everything is correct we will see the XCFrameworks on the Carthage/build folder and we can drag and drop to use it on our projects, since those are dynamic libraries they should be Embed $ Sign and should be also added to the respective markets where we are going to use it.

## Point of contact

- Primary contact - [@walmart-ios/glass-wellness-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-wellness-ios/members)
