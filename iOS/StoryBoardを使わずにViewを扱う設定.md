# StoryBoardを使わずにViewを扱う設定

## 環境
- Xcode 10.2.1
- Objective-C
- iPhone SE (simulator & real device)

## 手順
1. Single View Applicationでプロジジェクトを新規作成
2. Main.storyboard, LaunchScreen.storyboardの削除
  - 消さなくてもいい？
3. General => Development Info => Main Interfaceを空にする
4. AppDelegate.hを以下のように修正

```objc
[AppDelegate.h]

#import <UIKit/UIKit.h>
#import "ViewController.h"

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;
@property (strong,nonatomic) ViewController *viewController;

@end
```

5. AppDelegate.mを以下のように修正

```objc
[AppDelegate.m]

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.viewController = [[ViewController alloc] init];
    self.window.rootViewController = self.viewController;
    [self.window makeKeyAndVisible];
    
    return YES;

}
```

6. ViewController.mを以下のように修正
```objc
[ViewController.m]

#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSLog(@"viewDidLoad");
    
    CGRect rect = [self.view frame];
    UILabel* label = [[UILabel alloc] initWithFrame:rect];
    label.text = @"Hello View World!";
    self.view.backgroundColor = [UIColor whiteColor];
    [self.view addSubview:label];
}


@end
```

これでStoryBoardを使わずに画面上に「Hello World」と表示される。このときのフォルダ構成は以下

```
- MyFirstApp
  - AppDelegate.h
  - AppDelegate.m
  - ViewController.h
  - ViewController.m
  - Assets.scassets
  - Info.split
  - main.m

(testフォルダ省略)
```

## Buttonの追加
ViewController.mにボタンの処理を追加していく
```objc
[ViewController.m]

#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad{
  [super viewDidLoad];
  
  NSLog(@"viewDidLoad");
  
  // UILabelの作成
  CGRect rect = [self.view frame];
  UILabel* label = [[UILabel alloc] initWithFrame:rect];
  label.text = @"Hello View World!";
  self.view.backgroundColor = [UIColor whiteColor];
  [self.view addSubview:label];
  
  // ボタンの作成
  UIButton* button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
  [button setTitle:@"Touch me!" forState:UIControlStateNormal];
  [button sizeToFit];
  button.center = self.view.center;
  [button addTarget:self
          action:@selector(buttonDidPush)
          forControlEvents:UIControlEventTouchUpInside];
  [self.view addSubview:button];
}
@end
```

画面中央に「Touch Me!」ボタンが出現。押しても何も処理を追加していないのでなんかエラーになる

### Buttonの処理を実装
ボタンUIとメソッドのヒモ付は`action:@selector(<method>)`でできる。ここではbuttonDidPushメソッドを実装する

```objc
[ViewController.m]

// ボタンを押すと「Hello Button!」とアラートが出る
- (void)buttonDidPush {
    UIAlertView* alert = [[UIAlertView alloc] initWithTitle:nil message:@"Hello Button!" delegate:nil cancelButtonTitle:nil otherButtonTitles:@"OK",nil];
    [alert show];
}
```

## 参考サイト
- https://bluefish.orz.hm/sdoc/iphone_memo.html#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E4%BD%9C%E6%88%90
