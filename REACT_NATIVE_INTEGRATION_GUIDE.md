# React Native Integration & Bridge Specification
## Project: Security-First On-Device Biometric Verification & Active Liveness System
**Subsystem Profile:** Datalake 3.0 Cross-Platform Integration Bridge  
**Target Environment:** React Native (Android Native Java/Kotlin + iOS Swift/ObjC Bridge)

To facilitate cross-functional deployment in the existing React Native application for both Android and iOS devices, we present the structural bridging implementation.

---

## 1. Native Module Communication Pipeline
The communication pipeline is designed to cross the React Native asynchronous bridge:
1. JavaScript invokes `FaceLiveness.startLivenessChallenge(config)`.
2. The core platform-specific module opens the camera stream and handles ML-Kit offline processing.
3. Once the active liveness challenges (alignment, blink, smile, head turn) are passed, face biometric extraction is computed.
4. Results (embeddings or verification status) are resolved back to the JS layer as a Promise payload, or rejected if liveness fails or a duplicate is detected.

```
┌────────────────────────┐      JavaScript Bridge      ┌────────────────────────┐
│   React Native App     │ ──────────────────────────> │   Native Bridge Host   │
│  (UI & State Control)  │ <────────────────────────── │ (FaceLivenessModule.kt)│
└────────────────────────┘        Promise Resolve      └────────────────────────┘
                                                                   │
                                                                   ▼
                                                       ┌────────────────────────┐
                                                       │  On-Device ML Engine   │
                                                       │    (ML Kit Face SDK)   │
                                                       └────────────────────────┘
```

---

## 2. Android Bridge Source Code (Kotlin)
The following code snippet registers our fully offline Kotlin verification engine as a React Native `NativeModule` under the namespace `FaceLiveness`.

### A. Define the Module: `FaceLivenessModule.kt`
```kotlin
package com.example.bridge

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.Arguments
import android.content.Intent
import com.example.MainActivity

class FaceLivenessModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {

    private var mPromise: Promise? = null

    override fun getName(): String {
        return "FaceLiveness"
    }

    @ReactMethod
    fun startLivenessChallenge(mode: String, name: String, promise: Promise) {
        mPromise = promise
        val context = reactApplicationContext
        
        // Pass control to the specialized Jetpack Compose Screen inside MainActivity
        val intent = Intent(context, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK
            putExtra("MODE", mode) // "REGISTER" or "AUTHENTICATE"
            putExtra("NAME", name)
        }
        context.startActivity(intent)
        
        // Setup listener or use static callbacks to resolve promises
        FaceBridgeCallback.onResult = { isSuccess, message, similarity ->
            val result: WritableMap = Arguments.createMap().apply {
                putBoolean("success", isSuccess)
                putString("message", message)
                putDouble("similarity", similarity)
            }
            promise.resolve(result)
        }
        
        FaceBridgeCallback.onFailure = { error ->
            promise.reject("LIVENESS_FAILED", error)
        }
    }
}
```

### B. Register the Module Package: `FaceLivenessPackage.kt`
```kotlin
package com.example.bridge

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class FaceLivenessPackage : ReactPackage {
    override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
        return listOf(FaceLivenessModule(reactContext))
    }

    override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
        return emptyList()
    }
}
```

---

## 3. iOS Bridge Source Code (Objective-C & Swift)
To achieve parallel cross-platform behavior on iOS 12+ devices, we implement an Objective-C bridging header paired with a native Swift module.

### A. Define Objective-C Header: `FaceLivenessBridge.m`
```objc
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(FaceLiveness, NSObject)

RCT_EXTERN_METHOD(startLivenessChallenge:(NSString *)mode
                  name:(NSString *)name
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end
```

### B. Define Swift Implementation: `FaceLivenessBridge.swift`
```swift
import Foundation
import UIKit
import MLKitFaceDetection // iOS equivalent of MLKit On-Device Face

@objc(FaceLiveness)
class FaceLiveness: NSObject {
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return true
  }

  @objc
  func startLivenessChallenge(_ mode: String, name: String, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    DispatchQueue.main.async {
      // Launch native iOS view controller configured with the same dynamic active liveness flow
      let mainVC = UIApplication.shared.keyWindow?.rootViewController
      let livenessVC = NativeLivenessViewController()
      livenessVC.mode = mode
      livenessVC.targetName = name
      
      // Setup swift callback logic
      livenessVC.onCompletion = { result in
        resolve([
          "success": true,
          "message": result.message,
          "similarity": result.similarity
        ])
      }
      
      livenessVC.onCancel = {
        reject("LIVENESS_CANCELLED", "User closed scanner", nil)
      }
      
      mainVC?.present(livenessVC, animated: true, completion: nil)
    }
  }
}
```

---

## 4. JavaScript Wrapper Usage API
From the React Native JavaScript layer, invoking the fully offline facial recognition engine is simple and unified:

```typescript
import { NativeModules } from 'react-native';

const { FaceLiveness } = NativeModules;

export interface LivenessResponse {
  success: boolean;
  message: string;
  similarity: number;
}

/**
 * Executes a fully client-side on-device verification or enrollment flow.
 */
export async function runVerification(targetName: string): Promise<LivenessResponse> {
  try {
    const outcome: LivenessResponse = await FaceLiveness.startLivenessChallenge(
      "AUTHENTICATE",
      targetName
    );
    console.log("Biometric result:", outcome.message);
    return outcome;
  } catch (error) {
    console.error("Liveness failed:", error);
    throw error;
  }
}
```
