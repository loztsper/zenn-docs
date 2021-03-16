---
title: "イベントハンドラ内のerrorをError Boundaryでcatchさせる"
emoji: "🤗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript","react"]
published: true
---

イベントハンドラ内のthrow Errorを、Error Boundaryにcatchさせてみたくなった際に書いた短いメモ書きです。

## Error Boundaryとはそもそも

以下公式より引用
>error boundary は自身の子コンポーネントツリーで発生した JavaScript エラーをcatchし、エラーを記録し、クラッシュしたコンポーネントツリーの代わりにフォールバック用の UI を表示する React コンポーネントです。error boundary は配下のツリー全体のレンダー中、ライフサイクルメソッド内、およびコンストラクタ内で発生したエラーをcatchします。

https://ja.reactjs.org/docs/error-boundaries.html  

つまりJavaScript Errorをcatchしてくれる優れものです。  
ただ、非同期コードやイベントハンドラ内のErrorはcatchしません。
あくまでもrender中やhooks内のErrorに対してのみ反応します。

## 結論と解説


結論です。
```typescript
import { useCallback, useState } from "react";

export const useHandler = () => {
  const [_, setError] = useState<undefined | Error>(undefined);

  const throwError = useCallback((err: Error) => {
    setError(() => {
      throw err;
    });
  }, []);

  const onClick = useCallback(async () => {
    throwError(new Error("sample"));
  }, [throwError]);

  return { onClickButton };
};
```

stateをsetする関数の内部で、受け取ったErrorをthrowしています。これによりstateの変更途中にerrorがthrowされることになるため、結果的にError Boundaryがcatchするようになります。  
このままでは、onClickの途中でこのcustom hooksを呼んでいるComponentがunmountすると動作しなくなってしまうため、以下の部分をContextとして分離すると使用しやすくなるでしょう。

```typescript

  const [_, setError] = useState<undefined | Error>(undefined);

  const throwError = useCallback((err: Error) => {
    setError(() => {
      throw err;
    });
  }, []);
```

## おわりに

若干トリッキーな手法になるため万人にお勧めできる手法とは言えませんが、特定の機会においては活きると思います。