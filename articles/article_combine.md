---
title: "Swift CombineCocoaå¥é UIKitç·¨"
emoji: "ð¦"
type: "tech" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: ["iOS","Combine","Swift","ReactiveProgramming"]
published: false
---


# Combineã«å¥éãã¤ã¤UIKitãããã!
Combineãä½¿ã£ã¦,éåæå¦çãè²ãè©¦ãã¦ã¿ã
ä»åã¯,Switchã§ãã©ã°ãå¤æã§ããè¶³ãç®ã¢ããªãä½ãã¾ã
é¡æã¨ãã¦ã¯ãã¡ãã®åç»ãåèã«ãã¾ãã
<!-- TODO: ã¢ããªéå ´ã®èª²é¡ã®åç»ãæ¸ã -->
https://www.youtube.com/watch?v=dqYijANnyLc
# CombineCocoa
UIKitã®Componentsã«å¯¾ãã¦Combineãç°¡åã«ããã,CombineCocoaã¨ãããã¬ã¼ã ã¯ã¼ã¯ãæ´»ç¨ãã¦ããã¾ã!


# å®è£åã®æºå
## SPMããå°å¥

ä¸è¨ã®ãªã³ã¯ããããã±ã¼ã¸ãå°å¥ãã¾ã
https://github.com/CombineCommunity/CombineCocoa


## ãã¼ã¿ã®ããåãã®ç¢ºèª
ä»åã¯,å¥åããåå®¹ãã©ãã«ã«åæ ããããã¨
ã©ãã«ã®åå®¹ã,Switchã®ç¶æã«ãã£ã¦æ­£è² ãåè»¢ããããã«ãªãã¨ãããã¨ãæ¡ä»¶ã«ãã¾ã
![](/images/article_combine/marblediagram.png)
```mermaid
sequenceDiagram
    participant VC as ViewController 
    participant VM as ViewModel
    participant Model
    
    VC->>VM: Change numberSubject or switchSubject
    VM->>VM: Detect change from numberSubject and Call updat(String, Bool)
    VM->>Model: update(String, Bool)
    Model->>Model: assign new value to number(Published)
    Model->>VM: Imform update value
    VM->>VC: Assing new value on UIcomponents
```



1. æ°å¤ã®å¥åãPublishãã
2. Switchã®ç¶æãç¢ºèªãã
3. combineLatestã§ã©ã¡ããã®å¥åããã£ãéã«ããããã®å¤ãç¢ºèªãã¦Propertyã¨Labelãæ´æ°ãã


ä¸è¨ã¯,ã¡ãã£ã¨å¿ç¨çãªãªããããªã®ã§ã¹ã­ãããã¦ãå¤§ä¸å¤«ã§ã.
ããå®è·µçãªãã¯ããã¯ã¨ãã¦ææ¦ãããã¨ã«ãã¾ãã.
* ãã®ä»èæ®ããªãã¨è¡ããªãç¹
  * ãã¼ã¿ã®ããªãã¼ã·ã§ã³
    å¥åå¶éã¨ãã¦ä¸è¨ã®3ã¤ãç¢ºèªãã¾ã.è©²å½ããå ´åã«ã¯,Alertãè¡¨ç¤ºãã¾ã
    1. å¥åãç©º
    2. å¥åããã­ã¹ã
    3. åé ­ã«0ãå¥ãã
    4. å¥åã7æ¡æªæº

  * ãã¿ã³ã®Disable
    ä»åã¯,ãã¼ã¿ã®ããªãã¼ã·ã§ã³ãæºãããªãå ´åã«ã¯ãã¿ã³ãæå¹ã«ãªããªãããã«è¨­å®ãè¡ãã¾ã


# å®è£!!!

1. Proprtyã®å®ç¾©
ä»åã¯,å·¦å³ã®æ°å¤ããããã@Publsihed Propertyã¨ãã¦ä¿æãã¾ã
@Published ãä½¿ãã¨,æå®ãããã­ããã£ã¼ã®å¤ããã£ã¦ãããã¨ãã«ã¤ãã³ããçºè¡ããããã¨ã«ãªãã¾ã



2. subscriptionsã®å®ç¾©
3. Combine Publisherã®å®ç¾©

4. Buttonã«ãªãã¸ã§ã¯ããAssing
```swift
buttonSubscription
.recive(on: RunLoop.main)
.assign(to: \.isEnabled, on: signupButton)

```
* RuLoop.mainã¯ä½ããã¦ããã
SwiftLeeã®è¨äºåã§ã¯,ä¸è¨ã®ããã«ç¤ºããã¦ãã¾ã
> What is RunLoop.main?
> A RunLoop is a programmatic interface to objects that manage input sources, such as touches for an application. A RunLoop is created and managed by the system, whoâs also responsible for creating a RunLoop object for each thread object. The system is also responsible for creating the main run loop representing the main thread.

è¦ããã«RubLoopãã¿ãããªã©ã®ã¤ã³ãããããã­ã°ã©ããã£ãã¯ã«å¦çããã¤ã³ã¿ã¼ãã§ã¤ã¹ã¨ãããã¨ã§ãã.
ã¾ãããããããªãã§ãã­...



* assignã¯ä½ããã¦ããã®ã
ãµãã¹ã¯ãªãã·ã§ã³ã®çµæã«å¤æ´ããã£ãéã«,onã§æ¸¡ããObjectåã®ãã­ããã£ã¼ãto:ã§æå®ãã
æå®ã¯KeyPathã¨ããæ¹æ³ãä½¿ã£ã¦ãã
keypathã«ã¤ãã¦ã¯ãã¡ãã«ã¡ããã£ã¨ã¾ã¨ãã¾ããTweet






# åèè³æ
CombineCococaã®è§£èª¬è¨äº
https://dev.classmethod.jp/articles/combinecocoa/

Combineã®è§£èª¬è¨äº
https://www.youtube.com/watch?v=UvXvFa-k6dw
https://www.avanderlee.com/combine/runloop-main-vs-dispatchqueue-main/

RunLoopã®è§£èª¬è¨äº
https://prafullkumar77.medium.com/ios-run-loop-what-why-when-7febead400b7