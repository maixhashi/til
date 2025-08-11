# {{topic_name}} - ナレッジベース インデックス
<!-- このテンプレートはparams.ymlのパラメータを使用します -->
<!-- 使用方法:
     1. このファイルとparams.ymlをトピックディレクトリにコピー
     2. params.ymlを実際の値で更新
     3. {{variable}}プレースホルダーをparams.ymlの値で置換
-->

> [!NOTE]
> {{topic_name}}関連の知識のメインエントリーファイルです。
> TIL（Today I Learned）ファイルを目的別（Why/What/How）に整理・集約しています。

## 目次

<summary>目次</summary>
<details>
{{table-contents}} // {{topic-family}}と{{topic}}を抽出して目次表示
</details>

---


## TILs

{{each}} // {{topic-family}}, {{topic}}に関して繰り返し
### {{topic-family}}
#### {{topic}}
<summary>{{topic-family}} : {{topic}}</summary>
<details>
 - {{topic-family}}
   - {{topic}}
     - summarized-TIL
       - what
         - [{{filename_what}}]({{filelink_what}})
           - {{file-what-overview-description}} //...とは何か  
       - why
         - [{{filename_why}}]({{filelink_why}})
           - {{file-why-overview-description}} //なぜ...なのか
       - how
         - [{{filename_how}}]({{filelink_how}})
           - {{file-how-overview-description}} //どのように...なのか   
     - daily-TIL
       - what
         - [{{timestamp}}_{{filename_what}}]({{filelink_what}})
           - {{file-what-overview-description}} //...とは何か  
       - why
         - [{{timestamp}}_{{filename_why}}]({{filelink_why}})
           - {{file-why-overview-description}} //なぜ...なのか
       - how
         - [{{timestamp}}_{{filename_how}}]({{filelink_how}})
           - {{file-how-overview-description}} //どのように...なのか 
</details>
{{end}}