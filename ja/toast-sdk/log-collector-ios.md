## TOAST > TOAST SDK使用ガイド > TOAST Log & Crash > iOS

> [告知]
> TOAST SDK 0.13.0でarm64eアーキテクチャを使用する機器(iPhone XS、XR、XS Max、iPad Pros 3rd)で発生したクラッシュ集計、分析が可能です。

## Prerequisites

1. [TOAST SDK](./getting-started-ios)をインストールします。
2. [TOASTコンソール](https://console.cloud.toast.com)で、[Log & Crash Searchを有効化](https://docs.toast.com/ko/Analytics/Log%20&%20Crash%20Search/ko/console-guide/)します。
3. Log & Crash Searchで、[AppKeyを確認](https://docs.toast.com/ko/Analytics/Log%20&%20Crash%20Search/ko/console-guide/#appkey)します。

## TOAST Logger構成

* iOS用TOAST Logger SDKの構成は次のとおりです。

| Service  | Cocoapods Pod Name | Framework | Dependency | Build Settings |
| --- | --- | --- | --- | --- |
| TOAST Log & Crash | ToastLogger | ToastLogger.framework | [External & Optional]<br/> * CrashReporter.framework (Toast) |  |
| Mandatory   | ToastCore<br/>ToastCommon | ToastCore.framework<br/>ToastCommon.framework | | OTHER_LDFLAGS = (<br/>    "-ObjC",<br/>    "-lc++" <br/>); |

## TOAST Logger SDKをXcodeプロジェクトに適用

### 1. Cococapods適用

* Podfileを作成して、TOAST SDKに対するpodを追加します。

```podspec
platform :ios, '9.0'
use_frameworks!

target '{YOUR PROJECT TARGET NAME}' do
    pod 'ToastLogger'
end
```

### 2. バイナリをダウンロードしてTOAST SDK適用

#### Link Frameworks

* TOASTの[Downloads](../../../Download/#toast-sdk)ページで、全体iOS SDKをダウンロードできます。
* Xcode Projectに**ToastLogger.framework**、**ToastCore.framework**、**ToastCommon.framework**を追加します。
* TOAST LoggerのCrash Report機能を使用するには、一緒に配布される**CrashReporter.framework**もプロジェクトに追加する必要があります。
![linked_frameworks_logger](http://static.toastoven.net/toastcloud/sdk/ios/logger_link_frameworks_logger.png)

#### Project Settings

* **Build Settings**の**Other Linker Flags**に**-lc++**と**-ObjC**項目を追加します。
    * **Project Target > Build Settings > Linking > Other Linker Flags**
![other_linker_flags](http://static.toastoven.net/toastcloud/sdk/ios/overview_settings_flags.png)

* **CrashReporter.framewor**を直接ダウンロードするか、ビルドした場合は**Build Settings**の**Enable Bitcode**の値を**NO**に変更する必要があります。
    * **Project Target > Build Settings > Build Options > Enable Bitcode**
![enable_bitcode](http://static.toastoven.net/toastcloud/sdk/ios/overview_settings_bitcode.png)

> TOASTの[Downloads](../../../Download/#toast-sdk)ページでダウンロードしたCrashReporter.frameworkは、bitCodeをサポートします。

## TOAST Symbol Uploader 적용

### 프로젝트의 디버그 설정 변경
* 빌드 설정을 변경하여 프로젝트의 디버그 정보 형식을 변경해야합니다.
* Xcode -> Project Target -> Build Settings -> Debug Information Format -> Debug -> DWARF with dSYM File

### 개발 환경에서 Run Script를 사용하여 자동 업로드

* Xcode -> Project Target -> Build Phases -> + -> New Run Script Phase
* 표시되는 새 Run Script 섹션을 펼칩니다.
* Shell(셸) 필드 아래에 있는 스크립트 필드에서 새 실행 스크립트를 추가합니다.
```
if [ "${CONFIGURATION}" = "Debug" ]; then
    ${PODS_ROOT}/ToastSymbolUploader/toastcloud.sdk-*/run --app-key LOG_N_CRASH_SEARCH_DEV_APPKEY
fi
```
* LOG_N_CRASH_SEARCH_APPKEY에는 Log&Crash Search의 AppKey를 입력해야합니다.
* Run Script 섹션 하단의 Input Files에 dSYM의 기본 경로를 설정합니다.
    * ${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Resources/DWARF/${TARGET_NAME}

![symbol_uploader_script_pods_path](http://static.toastoven.net/toastcloud/sdk/ios/symbol_uploader_guide_script_pods_path.png)

### Symbol Uploader를 사용하여 직접 업로드

* SymbolUploader 사용법

```
USAGE: symbol-uploader -ak <ak> -pv <pv> [-sz <sz>] <path> [--verbose]

ARGUMENTS:
  <path>                  dSYM file path is must be entered.

OPTIONS:
  -ak, --app-key <ak>     [Log&Crash Search]'s AppKey must be entered.
  -pv, --project-version <pv>
                          Project version must be entered.
  -sz, --service-zone <sz>
                          You can choose between real, alpha, and demo. (default: real)
  --verbose               Show more debugging information
  -h, --help              Show help information.

```

* Xcode의 Run Script를 사용하지 않고 사용자가 원하는 시점에 아래와 같은 방법으로 SymbolUploader를 사용하여 직접 Symbol을 업로드 할 수 있습니다.

```
./SymbolUploader --app-key {APP_KEY} --project-version {CFBundleShortVersionString || MARKETING_VERSION} {symbol path(~/Project.dSYM)}
```

> `동일한 버전의 Symbol이 이미 업로드되어 있는 경우 SymbolUploader는 업로드되어 있는 Symbol을 제거하고 업로드를 수행합니다.`
> 이때 두 Symbol 파일의 `파일명이 다를 경우 업로드되어 있던 Symbol은 제거되지 않습니다.`
> Log & Crash Search 콘솔에서 업로드되어 있는 Symbol을 제거해야 합니다.
> https://console.toast.com/-> 조직 선택 -> 프로젝트 선택 -> Anaytics -> Log & Crash Search -> 설정 -> 심벌 파일

### CrashReport 使用時注意事項

* arm64eアーキテクチャを使用する機器のクラッシュ・分析のためにはTOAST Loggerと一緒に配布されるPLCrashReporterを使用しなければなりません。
      * TOASTの[Downloads](../../../Download/#toast-sdk)ページではない他の場所でダウンロードしたり、直接ビルドしたPLCrashReporterを使用する場合、arm64eアーキテクチャを使用する機器のクラッシュ分析が不可能です。

## TOAST Logger SDK初期化

* Log & Crash Searchで発行されたAppKeyを設定します。


### 初期化API仕様

``` objc
// 초기화
+ (void)initWithConfiguration:(ToastLoggerConfiguration *)configuration;
```

### 初期化プロセス例

```objc
ToastLoggerConfiguration *configuration = [ToastLoggerConfiguration configurationWithAppKey:@"YOUR_APP_KEY"];
[ToastLogger initWithConfiguration:configuration];
```

## ログ送信

* TOAST Loggerは、5つのレベルのログ送信関数を提供します。

### ログ送信API仕様

```objc
// DEBUG Level log
+ (void)debug:(NSString *)message;

// INFO Level log
+ (void)info:(NSString *)message;

// WARN Level log
+ (void)warn:(NSString *)message;

// ERROR Level log
+ (void)error:(NSString *)message;

// FATAL Level log
+ (void)fatal:(NSString *)message;
```

### ログ送信API使用例

```objc
[ToastLogger info:@"TOAST Log & Crash Search!"];
```

## ユーザー定義フィールド設定

* 希望するユーザー定義フィールドを設定します。
* ユーザー定義フィールドを設定すると、ログ送信APIを呼び出すたびに設定した値をログと一緒にサーバーに送信します。

### ユーザー定義フィールドAPI仕様

```objc
// ユーザー定義フィールド追加
+ (void)setUserFieldWithValue:(NSString *)value forKey:(NSString *)key;
```

* ユーザー定義フィールドは、**Log & Crash Search > ログ検索**をクリックした後、**ログ検索**画面の**選択したフィールド**に表示される値と同じです。

#### ユーザー定義フィールド制約事項

* すでに[予約されているフィールド](./log-collector-reserved-fields)は使用できません。
* フィールド名には'A-Z、a-z、0-9、-、_'を使用できます。最初の文字は'A-Z、a-z'のみ使用できます。
* フィールド名のスペースは、'_'に置換されます。


### ユーザー定義フィールド使用例
```objc
// ユーザー定義フィールド追加
[ToastLogger setUserFieldWithValue:@"USER_VALUE" forKey:@"USER_KEY"];
```

## クラッシュログの収集
* TOAST Loggerは、クラッシュ情報をログに送信する機能を提供します。
* TOAST Loggerを初期化する時、一緒に有効になり、使用するかを設定できます。
* クラッシュログを送信するには、PLCrashReporterを使用します。

### CrashReporter使用するかの設定
* CrashReporter機能は、基本的にTOAST Loggerを初期化する時に一緒に有効になります。
* TOAST Loggerを初期化する時、使用するかを設定できます。
* クラッシュログ送信機能を使用しない場合は、CrashReporter機能を無効にする必要があります。

> UserIDが設定されている場合、Log＆Crash Searchコンソールの`Crash User`セクションでユーザー固有のクラッシュ体験を確認できます。
> UserIDの設定は[開始する]（./getting-started-ios/#UserID設定）で確認できます。

#### CrashReporter有効化
```objc
// CrashReporter Enable Configuration
ToastLoggerConfiguration *configuration = [ToastLoggerConfiguration configurationWithAppKey:@"YOUR_APP_KEY"
                                                                        enableCrashReporter:YES];

[ToastLogger initWithConfiguration:configuration];
```
#### CrashReporter無効化
```objc
// CrashReporter Disable Configuration
ToastLoggerConfiguration *configuration = [ToastLoggerConfiguration configurationWithAppKey:@"YOUR_APP_KEY"
                                                                        enableCrashReporter:NO];

[ToastLogger initWithConfiguration:configuration];
```

## クラッシュ発生時に追加情報を設定して送信

* クラッシュ発生直後、追加情報を設定できます。
* setShouldReportCrashHandlerのBlockでユーザー定義フィールドを設定すると、正確にクラッシュが発生した時点に追加情報を設定できます。

### Data Adapter API仕様
```objc
+ (void)setShouldReportCrashHandler:(void (^)(void))handler;
```

### Data Adapter使用例

```objc
[ToastLogger setShouldReportCrashHandler:^{
  // ユーザー定義フィールドを通してCrashが発生した状況から得たい情報を一緒に送信
  // ユーザー定義フィールド追加
  [ToastLogger setUserFieldWithValue:@"USER_VALUE" forKey:@"USER_KEY"];

}];
```

## ログ送信後、追加作業進行

* Delegateを登録すると、ログ送信後に追加作業を進行できます。


### Delegate API仕様

```objc
+ (void)setDelegate:(id<ToastLoggerDelegate>) delegate;
@end
```

### Delegate API仕様

``` objc
@protocol ToastLoggerDelegate <NSObject>
@optional
// ログ送信成功
- (void)toastLogDidSuccess:(ToastLog *)log;

// ログ送信失敗
- (void)toastLogDidFail:(ToastLog *)log error:(NSError *)error;

// ネットワーク切断などの理由でログの送信に失敗した場合、再送信のためにSDK内部保存
- (void)toastLogDidSave:(ToastLog *)log;

// ログフィルタリング
- (void)toastLogDidFilter:(ToastLog *)log logFilter:(ToastLogFilter *)logFilter;
@end
```


### Delegate使用例

```objc
#import <ToastLogger/ToastLogger.h>

@interface AppDelegate () <UIApplicationDelegate, ToastLoggerDelegate>

@end


@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // ...

    // 초기화
    ToastLoggerConfiguration *configuration = [ToastLoggerConfiguration configurationWithAppKey:@"YOUR_APP_KEY"
                                                                            enableCrashReporter:YES];
    [ToastLogger initWithConfiguration:configuration];

    // Delegate 설정
    [[ToastLogger setDelegate:self];

    return YES;
}

#pragma mark - ToastLoggerDelegate
// ログ送信成功
- (void)toastLogDidSuccess:(ToastLog *)log {
      // ...
 }

// ログ送信失敗
- (void)toastLogDidFail:(ToastLog *)log error:(NSError *)error {
      // ...
}

// ネットワーク切断などの理由でログ送信に失敗した場合、再送信のためにSDK内部保存
- (void)toastLogDidSave:(ToastLog *)log {
      // ...
}

// ログフィルタリング
- (void)toastLogDidFilter:(ToastLog *)log logFilter:(ToastLogFilter *)logFilter {
      // ...
}

@end
```

## Network Insights
* Network Insightsは、コンソールに登録したURLを呼び出して、遅延時間とレスポンス値を測定します。これを活用して複数の国(デバイスの国コード基準)からの遅延時間とレスポンス値を測定できます。

> コンソールからNetwork Insights機能を有効にすると、TOAST Loggerを初期化する時、コンソールに登録したURLで1回要請します。

### Network Insights有効化

1. [TOAST Console](https://console.toast.com/)で**Log & Crash Search**サービスをクリックします。
2. **設定**メニューをクリックします。
3. **ログ送信設定**タブをクリックします。
4. **Network Insightsログ**を有効にします。

### URL設定

1. [TOAST Console](https://console.toast.com/)で**Log & Crash Search**サービスをクリックします。
2. **ネットワークインサイト**メニューをクリックします。
3. **URL設定**タブをクリックします。
4. 測定するにはURLを入力して**追加**ボタンをクリックします。
