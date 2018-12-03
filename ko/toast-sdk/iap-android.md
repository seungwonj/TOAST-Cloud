## TOAST > TOAST SDK 사용 가이드 > TOAST IAP > Android

## Prerequisites

1. [Install the TOAST SDK](./getting-started-android)
2. [TOAST 콘솔](https://console.cloud.toast.com)에서 [Mobile Service \> IAP를 활성화](https://docs.toast.com/ko/Mobile%20Service/IAP/ko/console-guide/)합니다.
3. IAP에서 [AppKey를 확인](https://docs.toast.com/ko/Mobile%20Service/IAP/ko/console-guide/#appkey)합니다.

### 개발환경
- Android Studio IDE 3.1.4 이상
- Android SDK Version은 4.0.3 (API Level 15) 이상
- Android Stuiod Gradle 설정

### 라이브러리 설정
- build.gradle에 라이브러리를 추가합니다.

```groovy
dependencies {
    implementation 'com.toast.android:toast-iap-google:0.12.0'
}
```

### 지원 마켓
- [Google](https://developer.android.com/google/play/billing/?hl=ko)
- [OneStore](https://dev.onestore.co.kr/devpoc/index.omp?selectLanguage=ko)

## 서비스 로그인

### 서비스 초기화
- TOAST IAP SDK를 사용하기 위해서는 ToastSdk를 초기화 해야 합니다. 초기화는 반드시 Application#onCreate에서 진행되어야 합니다.
- `초기화를 진행하지 않은 경우, TOAST SDK는 동작하지 않습니다.`

```java
public class MainApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        initializeToastSdk();
    }
    
    /**
    * ToastSdk 를 초기화합니다.
    *
    * ToastSdk 디버그 모드를 활성화하려면 ToastSdk.setDebugMode(boolean) 호출하여 true 로 설정합니다.
    * <pre>
    * {@code
    * ToastSdk.setDebugMode(true);
    * }
    * </pre>
    */
    private void initializeToastSdk() {
        ToastSdk.setDebugMode(true);
        ToastSdk.initialize(getApplicationContext());
    }
}
```

### 서비스 로그인
- ToastSDK의 모든 상품은 설정된 하나의 사용자 아이디를 사용합니다.
- 사용자 아이디가 설정되지 않은 경우, 결제가 진행되지 않습니다.
- 서비스 로그인 단계에는 사용자 아이디 설정, 미소비 결제 내역 조회, 활성화된 구독 상품 조회가 구현되는 것을 권장 합니다.
- 활성화된 구독 상품 조회의 경우 현재 Google 마켓만 지원하고 있습니다.
    

```java
public class MainActivity extends AppCompatActivity {
    /**
    * 앱이 로그인 되었을 때 ToastSdk.setUserId()을 호출하여 User Id를 설정해야합니다.
    * ToastIap는 ToastSdk에 설정된 User ID를 사용하여 결제를 진행합니다.
    *
    * User ID 설정 후 미소비 결제 내역 조회 및 활성화된 구독 상품 조회를 진행합니다.
    */
    private void onLogin() {
        // User Id 설정
        ToastSdk.setUserId(USER_ID);

        // 미소비 결제 내역 조회
        queryConsumablePurchases();

        // 활성화된 구독 상품 조회 (Google Only)
        queryActivatedPurchases();
    }

    /**
    * 미소비 결제 내역을 조회합니다.
    * 미소비된 결제는 Consume API를 사용하여 상품을 소비합니다.
    * Consume API : https://api-iap.cloud.toast.com/v1/service/consume
    */
    void queryConsumablePurchases() {
        ToastIap.queryConsumablePurchases(MainActivity.this, new IapService.PurchasesResponseListener() {
        @Override
        public void onPurchasesResponse(@NonNull IapResult result,
            @Nullable
            List<IapPurchase> purchases) {
                if (result.isSuccess()) {
                    // 성공
                    Log.i(TAG, result.toString());
                    if (purchases != null) {
                        Log.d(TAG, "Consumable purchase list.");
                        Log.d(TAG, purchases.toString());
                    }
                } else {
                    // 실패
                    Log.e(TAG, result.toString());
                }
            }
        });
    }

    /**
    * 활성화된 구독 상품 조회 (Google Only)
    */
    void queryActivatedPurchases() {
    ToastIap.queryActivatedPurchases(MainActivity.this, new IapService.PurchasesResponseListener() {
        @Override
        public void onPurchasesResponse(@NonNull IapResult result,
        @Nullable List<IapPurchase> purchases) {
            if (result.isSuccess()) {
            // 성공
            Log.i(TAG, result.toString());
                if (purchases != null) {
                    Log.d(TAG, "Consumable purchase list.");
                    Log.d(TAG, purchases.toString());
                }
            } else {
                // 실패
                Log.e(TAG, result.toString());
            }
        }
        });
    }
}
```


### 서비스 로그아웃
- 서비스를 종료하는 시점에 로그아웃을 구현합니다.

```java
public class MainActivity extends AppCompatActivity {
    /**
    * 로그아웃이 되었을 때 ToastSdk.setUserId(null)을 호출합니다.
    */
    private void onLogout() {
        ToastSdk.setUserId(null);
    }
}
```

```
Note : Google의 경우, 반드시 로그아웃을 구현해야 프로모션 코드가 리딤되었을 때 
잘못된 사용자 아이디로 구매가 진행되는 것을 방지할 수 있습니다.
```

## TOAST IAP SDK  초기화

### 초기화
- ToastIap를 초기화 합니다. 초기화는 반드시 Application#onCreate에서 진행되어야 합니다.
    
```java
public class MainApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        initializeToastIap();
        ...
    }

    /**
    * ToastIap 를 초기화합니다.
    */
    private void initializeToastIap() {
        ToastIapConfiguration configuration = ToastIapConfiguration.newBuilder(getApplicationContext())
        .setAppKey(APP_KEY)
        .setStoreCode(YOUR_STORE_CODE)
        .build();
        ToastIap.initialize(configuration);
    }
}
```

| 필드 | 설명 |
| -- | -- |
| APP_KEY | TOAST IAP에서 발급받은 AppKey |
| YOUR_STORE_CODE | 마켓 코드<br>IapStoreCode.GOOGLE_PLAY_STORE<br>IapStoreCode.ONE_STORE |

### 결제 콜백
- TOAST IAP의 모든 결제 결과는 PurchasesUpdatedListener를 등록하여 확인 가능합니다.

```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    /**
    * 인앱에서 소비성 상품, 구독, 프로모션 상품을 구매했을 때 결과를 통지합니다.
    */
    private PurchasesUpdatedListener mPurchaseUpdatedListener = new PurchasesUpdatedListener() {
        @Override
        public void onPurchasesUpdated(@NonNull List<IapPurchaseResult> purchaseResults) {
            Log.d(TAG, "Updated purchases.");
            for (IapPurchaseResult purchaseResult : purchaseResults) {
                if (purchaseResult.isSuccess()) {
                    // 성공
                    Log.i(TAG, purchaseResult.toString());
                    IapPurchase purchase = purchaseResult.getPurchase();
                    if (purchase != null) {
                    Log.d(TAG, purchase.toString());
                    }
                } else {
                // 실패
                Log.e(TAG, purchaseResult.toString());
                }
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // onCreate가 호출되었을 때 반드시 Listener를 등록합니다.
        registerPurchaseUpdatedListener(mPurchaseUpdatedListener);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // onDestroy()가 호출되었을 때 반드시 Listener를 제거합니다.
        unregisterPurchaseUpdatedListener(mPurchaseUpdatedListener);
    }

    void registerPurchaseUpdatedListener(@NonNull IapService.PurchasesUpdatedListener listener) {
        ToastIap.registerPurchasesUpdatedListener(listener);
    }

    void unregisterPurchaseUpdatedListener(@NonNull IapService.PurchasesUpdatedListener listener) {
        ToastIap.unregisterPurchasesUpdatedListener(listener);
    }
    ...
}

```

## 상품 목록 조회하기

### 상품 목록 조회
- TOAST IAP Console에 등록한 상품 목록을 조회합니다.

```java
/**
* 구매 가능한 상품을 조회합니다.
*
* productDetails : 구매 가능한 상품 목록
* invalidProducts : TOAST IAP 콘솔에 상품을 등록하였지만 구글에 등록되지 않은 상품
*/
void queryProductDetails() {
    ToastIap.queryProductDetails(MainActivity.this, new IapService.ProductDetailsResponseListener() {
    @Override
        public void onProductDetailsResponse(@NonNull IapResult result,
        @Nullable List<IapProductDetails> productDetails,
        @Nullable List<IapProduct> invalidProducts) {
            // 조회성공
            if (result.isSuccess()) {
                // 구매 가능한 상품 리스트
                if (productDetails != null) {
                    Log.d(TAG, productDetails.toString());
                }
                
                // 구매 불가능한 상품 리스트
                if (invalidProducts != null) {
                    Log.d(TAG, invalidProducts.toString());
                }
            } else {
                // 조회실패
                Log.e(TAG, result.toString());
            }
        }
    });
}
```
- IapProductDetails

| 필드 | 설명 |
| -- | -- |
| getProductId | 상품의 ID. 결제에 사용됨. |
| getProductSequence | 결제 고유 번호 |
| getPrice | 가격 |
| getLocalizedPrice | 현지 가격 |
| getPriceCurrencyCode | 통화 |
| getPriceAmountMicros | 1,000,000 단위 가격 |
| getFreeTrialPeriod | 무료 사용 기간 |
| getSubscriptionPeriod | 구독 기간 |
| getProductType | 상품 타입 |
| getProductTitle | 상품 타이틀 |
| getProductDescription | 상품 설명 |
| isActivated | 상품 사용 가능 여부 |

### 상품의 종류

- 현재 상품의 종류는 2가지로, 일회성 상품과 구독 상품을 지원합니다.
- 상품의 타입은 상품 목록 조회 결과의 IapProductDetails의 getProductType 함수를 사용하여 확인 가능합니다.

| 필드 | 설명 |
| -- | -- |
| CONSUMABLE | 소비 이전까지 관리되며, 소비 이후 사라지는 일회성 상품 |
| AUTO_RENEWABLE | 일정 기간마다 자동 결제되며, 활성화 기간동안 복원 가능한 구독 상품 |


## 상품 구매 하기
- 마켓에서 상품을 구매합니다. 
- productId는 상품 목록 조회 결과의 IapProductDetails의 getProductId 값을 사용합니다.

```java
/**
* 상품을 구매합니다.
*/
void launchPurchaseFlow(@NonNull Activity activity, @NonNull String productId) {
    IapPurchaseFlowParams params = IapPurchaseFlowParams.newBuilder()
    .setProductId(productId)
    .build();

    ToastIap.launchPurchaseFlow(activity, params);
}
```

## 구독 복원하기
- 현재 구독 상품은 Google Play Store만 지원합니다.

### 구독 복원
- 결제가 완료된 구독상품은, queryActivatedPurchases 통해 조회할 수 있습니다.
- 사용기간이 남아있는 경우, 계속해서 queryActivatedPurchases 통해 복원가능합니다.
- iOS에서 구독한 상품을 Android에서도 복원가능합니다. (서비스ID가 동일할 경우)

```java
/**
* 활성화된 구독 상품 조회
*/
void queryActivatedPurchases() {
    ToastIap.queryActivatedPurchases(MainActivity.this, new IapService.PurchasesResponseListener() {
    @Override
    public void onPurchasesResponse(@NonNull IapResult result,
    @Nullable List<IapPurchase> purchases) {
            if (result.isSuccess()) {
                // 성공
                Log.i(TAG, result.toString());
                if (purchases != null) {
                    Log.d(TAG, "Consumable purchase list.");
                    Log.d(TAG, purchases.toString());
                }
            } else {
                // 실패
                Log.e(TAG, result.toString());
            }
        }
    });
}
```

### 미소비 구매 내역 조회하기
- 일회성 상품의 경우 queryPurchases를 호출하여, 소비하지 않은 상품 정보를 조회합니다.

```java
/**
* 미소비 결제 내역을 조회합니다.
* 미소비된 결제는 https://api-iap.cloud.toast.com/v1/service/consume API 를 사용하여 상품을 소비합니다.
*/
void queryConsumablePurchases() {
    ToastIap.queryConsumablePurchases(MainActivity.this, new IapService.PurchasesResponseListener() {
        @Override
        public void onPurchasesResponse(@NonNull IapResult result,
        @Nullable List<IapPurchase> purchases) {
            if (result.isSuccess()) {
                // 성공
                Log.i(TAG, result.toString());
                if (purchases != null) {
                    Log.d(TAG, "Consumable purchase list.");
                    Log.d(TAG, purchases.toString());
                }
            } else {
                // 실패
                Log.e(TAG, result.toString());
            }
        }
    });
}
```


### 재처리
- 완료되지 못한 결제 프로세스를 재진행시킵니다.
- onResumed에 구현하는 것을 권장합니다.

```java
public class MainActivity extends AppCompatActivity {
    ...
    @Override
    protected void onResume() {
        super.onResume();
        // Google Play Store 에서 프로모션이 리딤된 결제 건을 처리하기 위해
        // Activity.onResume() 이 호출 되었을 때
        // queryConsumablePurchases()를 호출하여 프로모션 결제를 처리합니다.
        queryConsumablePurchases();
    }
}
```

### 에러 코드

## SDK

| RESULT | CODE | DESC |
| ------ | ---- | ---- |
| FEATURE_NOT_SUPPORTED | -2 | 요청한 기능은 지원하지 않습니다.<br>Requested feature is not supported. |
| SERVICE_DISCONNECTED | -1 | 스토어 서비스가 연결되지 않았습니다.<br>Store service is not connected. |
| OK | 0 | 성공<br>Success. |
| USER_CANCELED | 1 | 사용자 결제 취소<br>User canceled. |
| SERVICE_UNAVAILABLE | 2 | 네트워크 연결되지 않음<br>Network connection is down. |
| BILLING_UNAVAILABLE | 3 | 요청된 유형에 대해 Api Version이 지원되지 않습니다.<br>API version is not supported for the type requested. |
| PRODUCT_UNAVAILABLE | 4 | 요청한 상품을 사용할 수 없습니다.<br>Requested product is not available. |
| DEVELOPER_ERROR | 5 | 잘못된 인수가 API에 제공되었습니다. 개발 단계에서 발생하는 에러입니다.<br>Developer error. |
| ERROR | 6 | API 작업 중 치명적인 오류가 발생했습니다.<br>Fatal error during the API action. |
| PRODUCT_ALREADY_OWNED | 7 | 이미 소유한 상품이므로 구매하지 못했습니다.<br>Failure to purchase since item is already owned. |
| PRODUCT_NOT_OWNED | 8 | 소유하지 않은 상품이므로 소비하지 못합니다.<br>Failure to consume since item is not owned. |
| USER_ID_NOT_REGISTERED | 9 | 사용자 아이디가 등록되지 않음<br>User ID Is not registered. |
| UNDEFINED_ERROR | 9999 | 정의되지 않은 에러<br>Undefined error. |

## Mobill

| RESULT | CODE | DESC |
| ------ | ---- | ---- |
| INACTIVATED_APP | 101 | 활성화 되지 않은 앱입니다.<br>App is not active. |
| NETOWRK_NOT_CONNECTED | 102 | 네트워크가 연결되지 않았습니다.<br>Network not connected. |
| VERIFY_PURCHASE_FAILED | 103 | 결제 검증 실패에 실패했습니다. (1106)<br>Failure to verify purchase. |
| CONSUMED_PURCHASE | 104 | 구매가 이미 소비되었습니다. (5024)<br>Purchase already consumed. |
| REFUNDED_PURCHASE | 105 | 환불된 구매입니다. (5025)<br>Purchase already refunded. |

## OneStore

| RESULT | CODE | DESC |
| ------ | ---- | ---- |
| ONESTORE_NEED_LOGIN | 301 | 원스토어 서비스에 로그인되어 있지 않습니다.<br>OneStore service is not logged in. |
| ONESTORE_NEED_UPDATE | 302 | 원스토어 서비스가 업데이트 또는 설치되지 않았습니다.<br>OneStore service is not updated or installed. |
| ONESTORE_SECURITY_ERROR | 303 | 비정상 앱에서 결제를 요청하였습니다.<br>Abnormal purchase request. |
| ONESTORE_PURCHASE_FAILED | 304 | 결제 요청에 실패하였습니다.<br>Purchase request failed. |