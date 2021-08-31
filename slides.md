---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
---

# PHPUnitのはなし



---

# 話すこと

 - ユニットテストは何のためにやるか
 - メリット・デメリットは？
 - PHPUnitって結局何をやるもの？

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent; 
  -moz-text-fill-color: transparent;
}
</style>

---

# ユニットテストは何のためにやるか

 - *いま現在*、コードが実装者の意図通りに動くことを担保する
   - テストコードを見れば、どのように動く（と、実装者が思っている）のかがわかる
 - コードに変更があった場合、今まで通りに動くこと、あるいは動かないことを検証する


---

# メリット・デメリットは？

## メリット
<br>

 - デグレを**早期に**発見できる
   - リファクタリングの敷居が下がる🌟
 - カバレッジを追うことにより、**未検証コード**を排除できる
   - **未検証コード**。実務コードであっても意外とある😩
     - クリティカルなエラー処理（処理の途中でDBが落ちる等。書いてはあるが、検証されてなかったり。
     - 共有クラスや共通メソッド的なもので、「こんなこともあろうかと😏」で入れた機能やオプション
       - いますぐには使われないコードだから、検証もされないことが多い 
       - YAGNIでググろう 
     - ある機能を削除した際の削除漏れによる未使用コード
 - 設計力アップ（次ページへ続く
---

## メリット続き
<br>
<div grid="~ cols-2 gap-4">
<div>

ようし、じゃあ早速実務でテストコード書くか、<br>
とやってみると なぜかテストが書けない。<br>
それは、テストがしにくい設計になってしまっていることが多いから。<br>
例えばこんなの→
 - テストコード上でPersonsControllerをnewできる？
 - updateメソッドの引数$requestはどうやって用意する？
 - Responseがどうなっていたら登録が完了したとみなせる？
 - 登録エラーになった際、リダイレクトされたことをどう検証する？
</div>
<div>

```php
class PersonsController extends BaseController
{
    public function update(Request $request): Response
    {
        $id = $request->getPost('id');
        $name = $request->getPost('name');
        $email = $request->getPost('email');
        
        $person = PersonModel::find($id);
        $person->name = $name;
        $person->email = $email;
        if ($person->save() === false) {
            return redirect(
                '/persons', 
                '登録に失敗しました。'
            );
        }

        $view = new CompletedView();
        $view->message = '登録が完了しました！';
        return $view;
    }
}
```

</div>
</div>
---

<div grid="~ cols-2 gap-4">
<div>
↓こうなっていたら、登録処理のテストはできそう。

```php
class PersonsRepository
{
    public function update($id, $name, $email): bool
    {
        $person = PersonModel::find($id);
        $person->name = $name;
        $person->email = $email;
        return $person->save();
    }
}
```

</div>

<div>
こっちはテストしづらいまま。

```php
class PersonsController extends BaseController
{
    public function update(Request $request): Response
    {
        $id = $request->getPost('id');
        $name = $request->getPost('name');
        $email = $request->getPost('email');
        
        $repository = new PersonsRepository();
        if (!$repository->update($id, $name, $email)) {
            return redirect(
                '/persons', 
                '登録に失敗しました。'
            );
        }

        $view = new CompletedView();
        $view->message = '登録が完了しました！';
        return $view;
    }
}
```

</div>
</div>

---

## デメリット
実装工数が増える😡<br>
さらに、工数が増えるのに初回リリース時には恩恵に預かりづらいのも痛いところ。

---

# PHPUnitって結局何をやるもの？

下記みたいなプログラムをもっとスマートにやってくれるツール。

<div grid="~ cols-2 gap-4">
<div>

```php
<?php
$repository = new PersonsRepository();
if ($repository->update(
    1, 
    'USHIO', 
    'ushio@example.com') === false) {
    echo 'ERROR!'; // ここにきたらバグ
    return;
}
echo 'SUCCESS!'; // ここにきたら成功
```
</div>
<div>

```php
<?php
$repository = new PersonsRepository();
if ($repository->update(
    1, 
    'USHIO', 
    'hogehuga') === false) {
    echo 'ERROR!'; // ここにきたら成功（メールアドレスの形式が不正）
    return;
}
echo 'SUCCESS!'; // ここにきたらバグ
```

</div>
</div>
