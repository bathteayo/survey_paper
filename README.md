# 論文まとめ

## PDF to MD 
- 対象論文のPDFを[URL](https://colab.research.google.com/drive/1eUL_tIThqcDohTNuJ5TdDFpN3poossuw?usp=sharing#scrollTo=wBSUpZs12Bn6)からmd形式に変換する
  - 上部のランタイム->ランタイムのタイプを変更でハードウェア アクセラレータがGPU(T4 GPU)になっていることを確認．なっていない場合は変更．
  - 左側のファイルマークからファイルを開き，対象PDFをinput.pdfと名前を変更して配置(sample_dataと同じ階層)
  - 依存関係のインストールを実行
  - PDFをmmdフォーマットに変換を実行
  - 数式をtexフォーマットに変換を実行
- input.mdが作成されるので自分で数式を書いたりする必要がなくなる．

## まとめ方法
- proj-dagで@atsushi yanoに論文を訪ねる
- [論文調査リスト](https://docs.google.com/spreadsheets/d/1cud3mFUpiISnP2fHb7pNy4qh9WtxLikp/edit#gid=44443932)に対象論文を追加，作業予約をする．

未定
- `git checkout -b 発表年_学会名(短縮)_論文名` で新たにブランチを作成
- 新たにフォルダを作り，元論文とまとめファイルを配置する  
```
survey_paper  
├── 発表年_学会名(短縮)_論文名  
│   ├── 元論文.pdf  
│   ├── 発表年_学会名(短縮)_論文名.pdf
│   └── link.md (Notionへのリンクを記述)
├── README.md  
└── survey_template.md
```
未定　ここまで

- `PDF to MD`に従いPDFをマークダウン形式に自動変換する．
- `survey_template.md`に従って論文をまとめる．
  - `発表年_学会名(短縮)_論文名.pdf`と`発表年_学会名(短縮)_論文名.md`という名付け
- まとめ終わったらPRを出す
　- NotionをPDF化/md化したものを送る
- kobayuがチェックを行い修正があれば修正し，再度PRを出す
- 修正がなければ終了

## まとめ例
- [参考](https://ash-tulip-8e5.notion.site/CPU-Energy-Aware-Parallel-Real-Time-Scheduling-ECRTS-20-b3842a80f62e4fc48bbe5a9a6c22fdad)
