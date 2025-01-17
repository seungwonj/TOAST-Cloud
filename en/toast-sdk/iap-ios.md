## TOAST > User Guide for TOAST SDK > TOAST IAP > iOS

## Prerequisites

1. [Install TOAST SDK](./getting-started-ios).
2. [Enable Mobile Service \> IAP](https://docs.toast.com/ko/Mobile%20Service/IAP/ko/console-guide/) in [TOAST console](https://console.cloud.toast.com).
3. [Check AppKey ](https://docs.toast.com/ko/Mobile%20Service/IAP/ko/console-guide/#appkey)in IAP.

## Configuration of TOAST IAP

* TOAST Logger SDK for iOS is configured as follows.

| Service  | Cocoapods Pod Name | Framework | Dependency | Build Settings |
| --- | --- | --- | --- | --- |
| TOAST IAP | ToastIAP | ToastIAP.framework | * StoreKit.framework<br/><br/>[Optional]<br/> * libsqlite3.tdb | |
| Mandatory   | ToastCore<br/>ToastCommon | ToastCore.framework<br/>ToastCommon.framework | | OTHER_LDFLAGS = (<br/>    "-ObjC",<br/>    "-lc++" <br/>); |

## Apply TOAST SDK to Xcode Projects

### 1. Apply Cococapods

* Create a podfile to add pods to TOAST SDK.

```podspec
platform :ios, '9.0'
use_frameworks!

target '{YOUR PROJECT TARGET NAME}' do
    pod 'ToastIAP'
end
```

### 2. Apply TOAST SDK with Binary Downloads

#### Link Frameworks

* The entire iOS SDK can be downloaded from [Downloads](../../../Download/#toast-sdk) of TOAST.
* Add **ToastIAP.framework**, **ToastCore.framework**, **ToastCommon.framework, StoreKit.framework** to the Xcode Project.
* StoreKit.framework can be added in the following way.
![linked_storekit_frameworks](http://static.toastoven.net/toastcloud/sdk/ios/overview_link_frameworks_StoreKit.png)

![linked_frameworks_iap](http://static.toastoven.net/toastcloud/sdk/ios/iap_link_frameworks_iap.png)

#### Project Settings

* Add **-lc++** and **-ObjC** to **Other Linker Flags** at **Build Settings**.
    * **Project Target > Build Settings > Linking > Other Linker Flags**
![other_linker_flags](http://static.toastoven.net/toastcloud/sdk/ios/overview_settings_flags.png)

### Capabilities Setting

* To use TOAST IAP, you must enable the **In-App Purchase** item in Capabilities.
    * **Project Target > Capabilities > In-App Purchase**
![capabilities_iap](http://static.toastoven.net/toastcloud/sdk/ios/capability_iap.png)

## Service Login

* All TOAST SDK products(Log&Crash, IAP, Push, ...) are based on a same user ID.

### Login

* `Without user ID set, purchase, query of activated products, or query of consumed details are not available. `

``` objc
// Set user ID after service login is completed
[ToastSDK setUserID:@"INPUT_USER_ID"];
```

### Logout

``` objc
// Set user ID as nil after service logout is completed
[ToastSDK setUserID:nil];
```

## Initialize TOAST IAP SDK

* Set appkey issued from TOAST IAP.
* Reprocessing for uncompleted purchases is executed along with initialization.

### Specifications for Initialization API

``` objc
// Initialize
+ (void)initWithConfiguration:(ToastIAPConfiguration *)configuration;

// Set Delegate
+ (void)setDelegate:(nullable id<ToastInAppPurchaseDelegate>)delegate;

// Initialize and Set Delegate
+ (void)initWithConfiguration:(ToastIAPConfiguration *)configuration
                     delegate:(nullable id<ToastInAppPurchaseDelegate>)delegate;
```

### Specifications for Delegate API

* Register delegate to receive purchase result.
    * You can decide whether to proceed with the promotion payment in SDK or request payment directly when you want.
* Reprocessing results are not delegated, but are applied to the list of consumable products and the list of active subscription.
* `In order to receive delegate of purchase result, Delegate must be set before purchase of product.`

``` objc
@protocol ToastInAppPurchaseDelegate <NSObject>

// Purchase Succeeded
- (void)didReceivePurchaseResult:(ToastPurchaseResult *)purchase;

// Purchase Failed
- (void)didFailPurchaseProduct:(NSString *)productIdentifier withError:(NSError *)error;

@optional
// Select how to proceed with the promotion payment
- (BOOL)shouldAddStorePurchaseForProduct:(ToastProduct *)product API_AVAILABLE(ios(11.0));
@end
```

### Example of Initialization Procedure

``` objc
#import <UIKit/UIKit.h>
#import <ToastIAP/ToastIAP.h>

@interface ViewController () <ToastInAppPurchaseDelegate>

@end


@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // Initialize and Set Delegate
    ToastIAPConfiguration *configuration = [ToastIAPConfiguration configurationWithAppKey:@"INPUT_YOUE_APPKEY"];

    [ToastIAP initWithConfiguration:configuration delegate:self];
}

// Purchase Succeeded
- (void)didReceivePurchaseResult:(ToastPurchaseResult *)purchase {
    NSLog(@"Successfully purchased");
}

// Purchase Failed
- (void)didFailPurchaseProduct:(NSString *)productIdentifier withError:(NSError *)error {
    NSLog(@"Failed to purchase: %@", erorr);
}

// Select how to proceed with the promotion payment
- (BOOL)shouldAddStorePurchaseForProduct:(ToastProduct *)product {

    /*
    * return YES;
        * Make the requested promotion purchase in SDK.
        * purchase window will be printed after initialization and login.
    */
    return YES;

    /*
    * return NO;
        * Promotion purchase will be terminated.
        * After saving a product object, you can proceed with purchase with the stored object at any future point in time.
    */
    self.promotionProduct = product;
    return NO;
}
@end
```

## Query Product List

* IAP 콘솔에 등록된 상품이 [ToastProductResponse](./iap-ios/#toastproductresponse) 객체로 반환됩니다.
* IAP 콘솔에 등록된 상품 중 구매 가능한 상품은 products([ToastProduct](./iap-ios/#toastproduct))로 반환됩니다.
* IAP 콘솔에 등록된 상품 중 스토어(Apple)에서 상품 정보를 획득하지 못한 상품은 invalidProducts([ToastProduct](./iap-ios/#toastproduct))로 반환됩니다.

### Specifications for Product List Query API

``` objc
// Query Product List
+ (void)requestProductsWithCompletionHandler:(nullable void (^)(ToastProductsResponse * _Nullable response, NSError * _Nullable error))completionHandler;
@end
```

### Usage Example of Product List Query API

``` objc
[ToastIAP requestProductsWithCompletionHandler:^(ToastProductsResponse *response, NSError *error) {
    if (error == nil) {
        NSArray<ToastProduct *> *products = response.products;
        NSLog(@"Products : %@", products);

        // Failed to obtain product information from store
        NSArray<ToastProduct *> *invalidProducts = response.invalidProducts;
        NSLog(@"Invalid Products : %@", invalidProducts);

    } else {
        NSLog(@"Failed to request products: %@", error);
    }
}
```

### Product Types

| 상품명    | 상품타입             | 설명                                     |
| ------ | ---------------- | -------------------------------------- |
| Consumable Products | ToastProductTypeConsumable     | 소비 가능한 일회성 상품입니다. <br/>게임내 재화, 코인, 반복 구입 가능한 상품등에 사용할 수 있습니다. |
| Auto-Renewable Subscription Products  | ToastProductTypeAutoRenewableSubscription | 지정된 간격 및 가격으로 결제가 자동으로 반복되는 상품입니다, <br>잡지, 음악 스트리밍 접근 허용, 광고 제거등에 사용할 수 있습니다. |
| Auto-Renewable Consumable Subscription Products  | ToastProductTypeConsumableSubscription | 지정된 간격 및 가격으로 결제가 자동으로 반복되는 상품입니다. <br/>지정된 간격 및 가격으로 소비성 상품을 지급하고자 할 때 사용할 수 있습니다. |

> `Do not support Upgrades, Downgrades, and Modification for auto-renewable subscription products.`
> `Only one product must be registered to one subscription group.`

``` objc
typedef NS_ENUM(NSInteger, ToastProductType) {
    // Failed to Obtain Product Types
    ToastProductTypeUnknown = 0,

    // Consumable Products
    ToastProductTypeConsumable = 1,

    // Auto-Renewable Subscription Products
    ToastProductTypeAutoRenewableSubscription = 2,

    // Auto-Renewable Consumable Subscription Products
    ToastProductTypeConsumableSubscription = 3
};
```

## Purchase Product

* 구매 결과는 설정된 [ToastInAppPurchaseDelegate](./iap-ios/#toastinapppurchasedelegate)를 통해 전달됩니다.
* If an app is closed during purchase, or purchase is suspended due to network error, such purchase is reprocessed after the initialization of IAP SDK of the next app running.
* 구매 요청시 사용자 데이터 추가가 가능합니다.
* 사용자 데이터는 결제 결과(구매 성공 Delegate, 미소비 결제 내역, 활성화된 구독, 구매 복원)의 [ToastPurchaseResult](./iap-ios/#toastpurchaseresult) 객체에 포함되어 반환됩니다.
* 구매할 수 없는 상품이면 [ToastInAppPurchaseDelegate](./iap-ios/#toastinapppurchasedelegate)를 통해 구매 불가 상품임을 나타내는 오류가 전달됩니다.
* 상품 목록 조회 결과의 [ToastProduct](./iap-ios/#toastproduct) 객체 혹은 상품 아이디를 이용해 구매를 요청합니다.

### Specifications for Purchase Product API

``` objc
// Purchase product
+ (void)purchaseWithProduct:(ToastProduct *)product;

// Purchase product with payload
+ (void)purchaseWithProduct:(ToastProduct *)product payload:(NSString *)payload;

// Purchase product by identifier
+ (void)purchaseWithProductIdentifier:(NSString *)productIdentifier;

// Purchase product by identifier with payload
+ (void)purchaseWithProductIdentifier:(NSString *)productIdentifier payload:(NSString *)payload;
```

### Usage Example of Purchase Product API

``` objc
// Purchase product
[ToastIAP purchaseWithProduct:self.products[0] payload:@"DEVELOPER_PAYLOAD"];

// or

// Purchase product

[ToastIAP purchaseWithProductIdentifier:@"PRODUCT_IDENTIFIER" payload:@"DEVELOPER_PAYLOAD"];
```

## Query Activated Subscription List

* Query activated list of purchases for current user ID.
* Completely-paid subscription products(Auto-Renewal Subscription, Auto-Renewal Consumable Subscription) can be queried as long as usage period remains.
* Android subscription can also be queried for a same user ID.

### Specifications for Activated Subscription List API

``` objc
// Query Activated Subscription List
+ (void)requestActivePurchasesWithCompletionHandler:(nullable void (^)(NSArray<ToastPurchaseResult *> * _Nullable purchases, NSError * _Nullable error))completionHandler;
```

### Usage Example of Activated Subscription List Query API

``` objc
[ToastIAP requestActivePurchasesWithCompletionHandler:^(NSArray<ToastPurchaseResult *> *purchases, NSError *error) {
    if (error == nil) {
        for (ToastPurchaseResult *purchase in purchases) {
            // Activate access for subscription products
        }

    } else {
        NSLog(@"Failed to request active purchases : %@", error);
    }
}];
```

## Restore Purchases

* Restore the purchase history based on your AppStore account and apply it in the IAP console.
* Use if purchased subscription products are not viewed or activated.
* 만료된 결제건을 포함하여 복원된 결제건이 결과로 반환됩니다.
* 자동 갱신형 소비성 구독 상품의 경우 반영되지 않은 구매 내역이 존재할 경우 복원 후 미소비 구매 내역에서 조회 가능합니다.

### Specifications for Restoring Purchase API

``` objc
// Restore Purchase
+ (void)restoreWithCompletionHandler:(nullable void (^)(NSArray<ToastPurchaseResult *> * _Nullable purchases, NSError * _Nullable error))completionHandler;
```

### Usage Example of Restoring Purchase API

``` objc
[ToastIAP restoreWithCompletionHandler:^(NSArray<ToastPurchaseResult *> *purchases, NSError *error) {
    if (error == nil) {
        for (ToastPurchaseResult *purchase in purchases) {
            NSLog(@"Restored purchase : %@", purchase);
        }

    } else {
        NSLog(@"Failed to request restore : %@", error);
    }
}];
```

## Query Unconsumed Purchase List

* An consumable product must be processed as consumed after product is provided.
* 소비 처리되지 않은 구매 내역이 [ToastPurchaseResult](./iap-ios/#toastpurchaseresult) 객체로 반환됩니다.
* 자동 갱신형 소비성 구독 상품은 갱신 결제가 발생할 때마다 미소비 구매 내역에서 조회 가능합니다.

### Specifications for Unconsumed Purchase Query API

``` objc
// Query Unconsumed Purchases
+ (void)requestConsumablePurchasesWithCompletionHandler:(nullable void (^)(NSArray<ToastPurchaseResult *> * _Nullable purchases, NSError * _Nullable error))completionHandler;
```

### Usage Example of Unconsumed Purchase Query API

``` objc
[ToastIAP requestConsumablePurchasesWithCompletionHandler:^(NSArray<ToastPurchaseResult *> *purchases, NSError *error) {
    if (error == nil) {
        NSLog(@"Consumable Purchases : %@", purchases);

    } else {
        NSLog(@"Failed to request consumable : %@", error);
    }
}
```

## Consume Consumable Products

* Consumable products must be processed as consumed through REST API or Consume API of SDK, after products are provided.

### Specifications for Consumption API

``` objc
+ (void)consumeWithPurchaseResult:(ToastPurchaseResult *)result
                completionHandler:(nullable void (^)(NSError * _Nullable error))completionHandler;
```

### Usage Example of Consumption API

``` objc
// Query Unconsumed Purchases
[ToastIAP requestConsumablePurchasesWithCompletionHandler:^(NSArray<ToastPurchaseResult *> *purchases, NSError *error) {
    if (error == nil) {
        for (ToastPurchaseResult *purchaseResult in purchases) {
            //  Process as Product Provided
            // ...

            // Process as Consumed after Product is Provided
            [ToastIAP consumeWithPurchaseResult:purchaseResult
                              completionHandler:^(NSError *error) {
                                    if (error == nil) {
                                        NSLog(@"Successfully consumed");

                                    } else {
                                        NSLog(@"Failed to consume : %@", error);

                                        // Retreive Product Provided
                                        // ...
                                    }
                              }];
        }

    } else {
        NSLog(@"Failed to request consumable : %@", error);
    }
}
```

## Provide Page for Subscription Products

* For auto-renewable subscription products, users must be provided with a subscription management page.
> [Apple Guide](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/Subscriptions.html#//apple_ref/doc/uid/TP40008267-CH7-SW19)

* Without configuring a separate UI, call URL as below to display subscription management page.
### Connect to Subscription Management Page on Safari
```
https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/manageSubscriptions
```
```objc
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/manageSubscriptions"]];
```
#### Management page on Safari is called in the following order:
1. Safari Opens
2. Popup Shows: Want to open in iTunes Store?
3. iTunes Store Opens
4. Connected to subscription management page on a popup

>  `Safari` appears for Return to Previous App on top left on an iOS Device.


### Connect to Subscription Management Page on Scheme
```
itms-apps://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/manageSubscriptions
```

```objc
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"itms-apps://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/manageSubscriptions"]];
```
#### Management page on Scheme is called in the following order:
1. Subscription management page of App Store is directly connected with App-to-App call.

>  `Service App` appears for Return to Previous App on top left on an iOS device.



## Remain Compatible with (old) IAP SDK

To remain compatible with (old) IAP SDK, reprocessing is supported for incomplete purchases created by (old) IAP SDK.
>To enable compatibility with (old) IAP SDK, additionally link `sqlite3 Library(libsqlite3.tdb)`.

![linked_sqlite3](http://static.toastoven.net/toastcloud/sdk/ios/iap_link_sqlite3.png)

### Specifications for Reprocessing Incomplete Purchase API

``` objc
+ (void)processesIncompletePurchasesWithCompletionHandler:(nullable void (^)(NSArray <ToastPurchaseResult *> * _Nullable results, NSError * _Nullable error))completionHandler;
```

### Usage Example of Reprocessing Incomplete Purchase

``` objc
// Request for Reprocessing Incomplete Purchase
[ToastIAP processesIncompletePurchasesWithCompletionHandler:^(NSArray<ToastPurchaseResult *> *results, NSError *error) {
    if (error == nil) {
        for (ToastPurchaseResult *purchaseResult in results) {
            // Process as Product Provided
            // ...

            // Process as Consumed after Product Provided
            [ToastIAP consumeWithPurchaseResult:purchaseResult
                              completionHandler:^(NSError *error) {
                                    if (error == nil) {
                                        NSLog(@"Successfully consumed");

                                    } else {
                                        NSLog(@"Failed to consume : %@", error);

                                        // Retrieve Product Provided
                                        // ...
                                    }
                              }];
        }

    } else {
        NSLog(@"Failed to process incomplete purchases : %@", error);
    }
}];
```

## TOAST IAP Class Reference

### ToastIAPConfiguration

TOAST IAP 초기화 메소드의 파라미터로 사용되는 인앱 결제 설정 정보입니다.

```objc
@interface ToastIAPConfiguration : NSObject <NSCoding, NSCopying>

// IAP 서비스 앱 키
@property (nonatomic, copy, readonly) NSString *appKey;
// 서비스 존
@property (nonatomic) ToastServiceZone serviceZone;

+ (instancetype)configurationWithAppKey:(NSString *)appKey;

- (instancetype)initWithAppKey:(NSString *)appKey
NS_SWIFT_NAME(init(appKey:));

@end
```

## ToastInAppPurchaseDelegate

결제 결과를 통지받고 프로모션 결제의 수행 방식을 설정 할 수 있습니다.

```objc
@protocol ToastInAppPurchaseDelegate <NSObject>

// 결제 성공
- (void)didReceivePurchaseResult:(ToastPurchaseResult *)purchase
NS_SWIFT_NAME(didReceivePurchase(purchase:));

// 결제 실패
- (void)didFailPurchaseProduct:(NSString *)productIdentifier withError:(NSError *)error
NS_SWIFT_NAME(didFailPurchase(productIdentifier:error:));

@optional
// 프로모션 결제 진행 방법 선택
- (BOOL)shouldAddStorePurchaseForProduct:(ToastProduct *)product API_AVAILABLE(ios(11.0));

@end
```

## ToastProductResponse

상품 목록 정보를 확인 할 수 있습니다.

```objc
@interface ToastProductsResponse : NSObject <NSCoding, NSCopying>

// IAP 콘솔과 스토어(Apple)에 등록되어 있는 결제에 사용할 수 있는 상품 목록
@property (nonatomic, copy, readonly) NSArray<ToastProduct *> *products;
// 스토어(Apple)에서 상품 정보를 획득하지 못한 상품 목록
@property (nonatomic, copy, readonly) NSArray<ToastProduct *> *invalidProducts;

@end
```

## ToastProduct

TOAST IAP 콘솔에 등록된 상품의 정보를 확인할 수 있습니다.

```objc
@interface ToastProduct : NSObject <NSCoding, NSCopying>

// 상품의 ID
@property (nonatomic, copy, readonly) NSString *productIdentifier;
// 상품 고유 번호
@property (nonatomic, readonly) long productSeq;
// 상품 이름 (IAP Console)
@property (nonatomic, copy, readonly, nullable) NSString *productName;
// 상품 유형
@property (nonatomic, readonly) ToastProductType productType;
// 가격
@property (nonatomic, copy, readonly, nullable) NSDecimalNumber *price;
// 통화
@property (nonatomic, copy, readonly, nullable) NSString *currency;
// 현지 상품 이름 (AppStoreConnect)
@property (nonatomic, copy, readonly, nullable) NSString *localizedTitle;
// 현지 상품 설명 (AppStoreConnect)
@property (nonatomic, copy, readonly, nullable) NSString *localizedDescription;
// 현지 가격
@property (nonatomic, copy, readonly, nullable) NSString *localizedPrice;
// 상품 활성화 여부
@property (nonatomic, readonly, getter=isActive) BOOL active;
// 스토어 코드 "AS"
@property (nonatomic, copy, readonly) NSString *storeCode;

@end
```

## ToastPurchaseResult

결제 정보를 확인할 수 있습니다.

```objc
@interface ToastPurchaseResult : NSObject <NSCoding, NSCopying>

// 사용자 ID
@property (nonatomic, copy, readonly) NSString *userID;
// 스토어 코드 "AS"
@property (nonatomic, copy, readonly) NSString *storeCode;
// 상품의 ID
@property (nonatomic, copy, readonly) NSString *productIdentifier;
// 상품 고유 번호
@property (nonatomic, readonly) long productSeq;
// 상품 유형
@property (nonatomic, readonly) ToastProductType productType;
// 가격
@property (nonatomic, copy, readonly) NSDecimalNumber *price;
// 통화
@property (nonatomic, copy, readonly) NSString *currency;
// 결제 고유 번호결제 ID
@property (nonatomic, copy, readonly) NSString *paymentSeq;
// 소비에 사용되는 토큰
@property (nonatomic, copy, readonly) NSString *accessToken;
// 결제 ID
@property (nonatomic, copy, readonly) NSString *transactionIdentifier;
// 원본 결제 ID
@property (nonatomic, copy, readonly, nullable) NSString *originalTransactionIdentifier;
// 상품 구매 시간
@property (nonatomic, readonly) NSTimeInterval purchaseTime;
// 구독 상품의 만료 시간
@property (nonatomic, readonly) NSTimeInterval expiryTime;
// 프로모션 결제 여부
@property (nonatomic, readonly, getter=isStorePayment) BOOL storePayment;
// 사용자 데이터
@property (nonatomic, readonly, copy, nullable) NSString *payload;

@end
```

### Error Codes

```objc
// IAP Error
static NSString *const ToastIAPErrorDomain = @"com.toast.iap";

typedef NS_ENUM(NSUInteger, ToastIAPErrorCode) {
    ToastIAPErrorUnknown = 0,                       // Unknown
    ToastIAPErrorNotInitialized = 1,                // Not Initialized
    ToastIAPErrorStoreNotAvailable = 2,             // Store is unavailable
    ToastIAPErrorProductNotAvailable = 3,           // Failed to get product information
    ToastIAPErrorProductInvalid = 4,                // Inconsistency of IDs between original payment and current product
    ToastIAPErrorAlreadyOwned = 5,                  // Product is already owned
    ToastIAPErrorAlreadyInProgress = 6,             // Request is already processing
    ToastIAPErrorUserInvalid = 7,                   // Inconsistency of IDs between current user and paid user
    ToastIAPErrorPaymentInvalid = 8,                // Failed to get futher payment information (ApplicationUsername)
    ToastIAPErrorPaymentCancelled = 9,              // Store payment cancelled
    ToastIAPErrorPaymentFailed = 10,                // Store payment failed
    ToastIAPErrorVerifyFailed = 11,                 // Receipt verification failed
    ToastIAPErrorChangePurchaseStatusFailed = 12,   // Change of purchase status failed
    ToastIAPErrorPurchaseStatusInvalid = 13,        // Unavailable to purchase
    ToastIAPErrorExpired = 14,                      // Subscription expired
    ToastIAPErrorRenewalPaymentNotFound = 15,       // Renewal payment information not found in receipt
    ToastIAPErrorRestoreFailed = 16,                // Failed to restore
    ToastIAPErrorPaymentNotAvailable = 17,          // Status of purchase inoperative (e.g. setting purchase restrictions in app)
    ToastIAPErrorPurchaseLimitExceeded = 18,        // 월 구매 한도 초과
};

// Network Error
static NSString *const ToastHttpErrorDomain = @"com.toast.http";

typedef NS_ENUM(NSUInteger, ToastHttpErrorCode) {
    ToastHttpErrorNetworkNotAvailable = 100,        // Network is unavailable
    ToastHttpErrorRequestFailed = 101,              // HTTP Status Code is not 200 or can not read request
    ToastHttpErrorRequestTimeout = 102,             // Timeout
    ToastHttpErrorRequestInvalid = 103,             // Request is invalid
    ToastHttpErrorURLInvalid = 104,                 // URL is invalid
    ToastHttpErrorResponseInvalid = 105,            // Response is invalid
    ToastHttpErrorAlreadyInprogress = 106,          // Request is already in progress
    ToastHttpErrorRequiresSecureConnection = 107,   // Do not set Allow Arbitrary Loads
};
```
