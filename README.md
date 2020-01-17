# Compare performance of event phase

# About
- JavaScript の イベント は`capture phase`で行われるものと、`bubble phase`で行われるものがある
- この２つの phase のうち`bubble phase`が一般的で default になっている
- ここで`bubble phase`を利用してイベントを設定するのと、一つ一つの要素にイベントを設定していくのではどっちがパフォーマンスが高いのかを比較したいと考えた
- このレポジトリでは、計測の結果と計測の方法、利用したコードなどを載せている

# How to measure performance
- 計測の方法は`Chrome`の`DevTools`を使用した
- `Chrome`の`DevTools`の`performanceタブ`で計測し、`scripting`の時間を集計して、平均で比較する
- 計測の範囲はリロードをして、ローディングが始まってからrenderingが終わるまでの間である
- 各5回の結果の集計を各アイテム数で行った(10000 items, 30000 items, 50000 items, 100000 items)

# code
### without bubble phase
```js
  const list = document.getElementById("list");

  for(let i = 0; i < 10000; i++) {
    const item = document.createElement('li');
    item.id = `item-${i}`;
    item.textContent = `item ${i}`;
    item.addEventListener("click", (e) => {
       console.log(e.target.textContent);
    });
    list.appendChild(item);
  }
```

### bubble phase
```js
  const list = document.getElementById("list");

  for(let i = 0; i < 10000; i++) {
    const item = document.createElement('li');
    item.id = `item-${i}`;
    item.textContent = `item ${i}`;
    list.appendChild(item);
  }

  list.addEventListener("click", (e) => {
    if(!e.target || e.target.nodeName !== "LI") {
      return;
    }
    const selectedItem =document.getElementById(e.target.id);
    console.log(selectedItem.textContent);
  });
```

# Result

| item length | bubble phase (rendering / scripting) | without (rendering / scripting) |
| :---: | :---: | :---: |
| 10000 items | 50ms | 90ms |
| 30000 items | 138ms | 272ms |
| 50000 items | 174ms | 479ms |
| 100000 items | 402ms | 592ms |

- `scripting`の結果を見ると、`bubble phase`を利用した処理の方が速かった
- これは関数の生成回数や、代入の回数、メモリの利用量などが減ったことによるものだと考えられる
- `addEventListener`はメモリにも影響してくるので、メモリ利用量なども計測してみたい
