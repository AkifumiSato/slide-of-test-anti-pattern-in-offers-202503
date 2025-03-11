---
theme: default
# https://unsplash.com/ja/%E5%86%99%E7%9C%9F/%E6%97%A5%E9%A3%9F%E3%81%AE%E3%83%87%E3%82%B8%E3%82%BF%E3%83%AB%E5%A3%81%E7%B4%99-_ok8uVzL2gI
background: /background.png
class: text-center
highlighter: shiki
lineNumbers: false
colorSchema: "dark"
drawings:
  persist: false
transition: slide-left
title: 過剰テスト中毒とエラーテスト欠乏症 - UIテスト二大疾病の根治療法
mdc: true
---

# 過剰テスト中毒と<br>エラーテスト欠乏症

in Offers 202503

UIテスト二大疾病の根治療法

---
layout: two-cols
---

# Profile

- 名前: 佐藤 昭文（あっきー）
- ID: [akfm_sato](https://x.com/akfm_sato)
- 仕事: フロントエンドエキスパート
- 活動
  - 執筆: e.g. [Next.jsの考え方](https://zenn.dev/akfm/books/nextjs-basic-principle)
  - OSS: [`location-state`](https://github.com/recruit-tech/location-state)

::right::

<div class="pt-10 flex justify-center">
  <img src="https://avatars.githubusercontent.com/u/25711332?v=4" width="200" height="200">
</div>

---

# 今日のテーマ: UIテスト二大疾病

単体テストにおける2つの陥りやすいアンチパターンと、その対処法について

- _過剰テスト中毒_
- _エラーテスト欠乏症_

---
layout: section
---

# 過剰テスト中毒

単体テスト初学者が最初に陥るアンチパターン

---

# 過剰テスト中毒とは

書けるテストをともかく書いてると、後々苦しくなってくる

- 過剰テスト: テストとしての価値が低いもの
- 過剰テスト中毒: 過剰テストによってメンテ工数が増えてしまっている状態

<p>テストとしての<span class="font-bold" v-mark="{ at: 1, color: 'red', type: 'underline'}">価値が低いものを排除</span>することが重要</p>

---
transition: fade
---

# 過剰テストの例

ページコンポーネントの実装

```tsx
function RootPage() {
  // ...

  return (
    <div>
      <h1>Hello, World</h1>
      ...
      <button type="submit">問い合わせ</button>
    </div>
  );
}

export default HomePage;
```

---
transition: fade
---

# 過剰テストの例

過剰テストと考えられるテストケース

```tsx
test("ページタイトルに'Hello, World'が存在すること", () => {
  render(<RootPage />);

  expect(
    screen.getByRole("heading", {
      name: "Hello, World",
    }),
  ).toBeInTheDocument();
});
```

<p>あまりに自明なテストは<span class="font-bold" v-mark="{ at: 1, color: 'red', type: 'underline'}">バグを拾う確率が低い</span></p>

---

# 良いテストの例

ボタン押下時の挙動はアプリケーションにおいても重要性が高く、デグレする可能性も高い部類

```tsx
test("問い合わせ完了後に完了画面へ遷移すること", () => {
  // ...

  await user.click(
    screen.getByRole("button", {
      name: "問い合わせ",
    }),
  );

  expect(apiReq).toHaveBeenCalledTimes(1);
  expect(apiReq).toHaveBeenLastCalledWith({
    // ...
  });
  expect(
    await screen.findByRole("heading", {
      name: "問い合わせが完了しました",
    }),
  ).toBeInTheDocument();
});
```

<p>formのsubmit時処理のテストは<span class="font-bold" v-mark="{ at: 1, color: 'red', type: 'underline'}">バグを拾う確率が比較的高い</span></p>

---

# 過剰テストを判断する審美眼

良い単体テストとは、以下の4つの指標によって判断することが可能

<div class="flex justify-between">

- 退行に対する保護
- リファクタリングへの耐性
- 迅速なフィードバック
- 保守のしやすさ

<div class="flex w-100">
  <img src="https://book.mynavi.jp/files/topics/134252_ext_06_0.jpg?v=1670578534" width="300" height="386">
</div>

</div>

---
layout: section
---

# エラーテスト欠乏症

単体テスト初学者が見落としやすいテスト観点

---

# エラーテスト欠乏症

多くの人が正常系のテストを書くが、異常系のテストは抜けがち

- エラーテスト: 通信の失敗などに代表される、エラー時UIに関するテスト
- エラーテスト欠乏症: エラーテストが一切ないもしくは足りてない状態

<p>ユーザーにとってエラー時の挙動は<span class="font-bold" v-mark="{ at: 1, color: 'red', type: 'underline'}">当たり前品質の観点</span>で重要</p>

<div class="flex justify-center w-full pt-10">
  <img src="https://res.cloudinary.com/zenn/image/fetch/s--ITHBND-p--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/0ed3bc6193f3803c283653f0.png%3Fsha%3D3ef097393196970ef8148e3859f16038130b1caf" alt="狩野モデル" width="374" height="202">
</div>

---

# エラーテストを含めたテスト観点

Todoアプリにおける更新画面のテスト観点例

![テスト観点](/error-test-list.png)

<p>想定できるエラーを可能な限り<span class="font-bold" v-mark="{ at: 1, color: 'red', type: 'underline'}">網羅的にテスト</span>することが重要</p>

---

# エラーテストの例

Todoアプリにおける更新画面のテスト実装例

```tsx
describe("Todo取得通信でエラー", () => {
  test("ネットワークエラー時、エラーメッセージが表示されること", async () => {
    // Arrange
    const apiRequestCall = jest.fn();
    server.use(
      http.get("/api/todos", (req, res, ctx) => {
        return apiRequestCall(HttpResponse.error());
      }),
    );
    // Act
    render(<TodoApp />);
    // Assert
    expect(await screen.findByRole("alert")).toHaveTextContent(
      "通信エラーが発生しました。通信環境をご確認の上、再度お試しください。",
    );
    expect(apiRequestCall).toHaveBeenCalledTimes(1);
  });
});
```

---
layout: section
---

# まとめ

UIテスト二大疾病の根治療法

---

# まとめ

テスト初学者が陥りやすいアンチパターンを念頭に、良いテストを目指しましょう

- *過剰テスト中毒*の対処: テストの価値を見極め、良いテストに絞って書くことを意識しましょう
- *エラーテスト欠乏症*の対処: エラーパターンを列挙して、網羅的にテストしましょう

---
layout: section
---

# End
