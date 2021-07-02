# revert-test

## やりたいこと
revertのrevertの検証

## 前提
* develop -> staging -> master へ反映していくブランチ運用を取っている

## 想定する状況
* 機能Aをmasterまでリリース
* リリース直後に障害が起き、念のため直近のリリースをrevertした (X)
  * (手順1)develop -> stagingへマージするPRをrevert
  * (手順2)staging -> masterへマージ (=直近のリリースをrevertして本番環境を切り戻す)
* 原因を調査した結果、機能Aには問題がなく、過去の別コード(機能B)に問題があった
* 機能Bを修正し、リリースした(=develop -> staging -> masterへ反映。改修箇所は機能Aと別のファイルにある)
* Xで行った、機能Aのrevertをrevertし、機能Aを再リリースしたい
* 更に言うと、機能Aをちょっとだけ修正した上で再リリースしたい
  ...んだけど、適切なやり方が分からないため実験しよう!

## 実験
```
masterに1〜7を追加 -> push
developで1を編集 -> push
developで2を編集 -> push
developをstagingにmerge(#1)
developで3を編集 -> push  // revertのrevert対象
developをstagingにmerge(#2)
developで4を編集 -> push
developで5を編集 -> push
staging上で「developをstagingにmerge(#2)」をrevert(GitHubのPR上のボタンを押す) (X)
developをstagingにmerge(#2)
-- ここまでで、現状を再現できた
-- 複数のやり方を試すため、この時点のdevelop/stagingのクローンとしてdevelop-a, develop-b, staging-a, staging-bを作成


A: staging->developへリバートコミットを反映後、develop上でmerge(参考: https://tic40.hatenablog.com/entry/2019/04/09/003000)
  staging-aをdevelop-aへmerge  // 全コミットを逆反映。差分はrevert分のみ。
  develop上でXをrevert         // <- エラーが出てrevertできない

B: staging上でリバートのリバートを行う
  staging上でXをrevert
  develop上で6を編集 -> push
  develop上で3を編集 -> push  // こいつをstagingにマージする時にコンフリクトしないかが焦点
  developをstagingにmerge(#4) // <- こいつはコンフリクトしない
```

## 結論
パターンBでやればOK
