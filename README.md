# RegularExpressionLibrary
自家製正規表現パターンマッチエンジン

C++、ATL、STLでCOM仕様ライブラリ、テスト用にC#、forms

文字列のパターンマッチのみならずクラス配列とかをマッチングさせようと試みた

動作としては、パターンをコンパイルして定数(定文字?)、メタ文字、繰り返し等の各パターンにマッチさせてポインタを進めるイテレーター群を作り、すべてのイテレーターがtrueを返せばマッチ成功となる感じ

一定のバイトサイズをCOMに与え、そのサイズに合わせたパターンマッチエンジンをNEWして返す

とりあえず1,2,4バイトとかでテストしていたらクラスも8バイトぐらいのハッシュで代用できるんじゃね？との考えに至り気力がなえた。

当時BoostRegexにかなり触発されてアイデアを頂いている

パターン構文(?...)を拡張してイベントを発生させることができる

IF構文、Switch構文、マッチが到達したイベント、等々

イベントを拾ってパース中のステート変更とかを制御することを想定した

この機能でリッチテキストファイル(.RTF)のパーサーを作ってみた

tree.h						ツリー構造のオブジェクトコンテナ　中身はstd::vectorで各オブジェクトにインデント情報をつけてツリー構造を表現　大昔にTreeViewコントロールを自作する過程で作ったもの　ツリー構造を表現するときはこれを使うのがマイルール

RegExp.h					対象のマッチイテレータークラスの定義　このイテレーターの組み合わせでマッチパターンを表現する

RegExp_Builder.h			Boostで言うところの定義済みパーサの定義

文字列のパターン構文

	...						マッチパターン
	{R}						呼び出し更新
	.|.						ブランチ構文　|(or)接続されたマッチパターン
	～						定数指定構文　(\d+|0[xX]\d+)?　未指定で0とする
	(?(?～)...)				bool func(int 定数)イベント　返値でマッチパターンに進む
	(?(#～)...)				void func(int 定数)イベント　到達イベントを起こし、マッチパターンに進む
	(?(<～:.|.)...)			Switch構文　int func(int 定数)イベント　返値で各ブランチを選択し、マッチしたらマッチパターンに進む
	(?(>～).|.)				Select構文　void func(int 定数, int 各ブランチでマッチしたインデックス)イベント
	(?(@～:{R})...)			For構文　{R}がマッチしている間マッチパターンに進む
	(?(*{R})...)			While構文　{R}がマッチしている間マッチパターンに進む
	(?(+{R})...)			Do～While構文　一度以上のWhile
	(?(|...)...)			IF構文　マッチ結果をイベント
	(?(=...)...)			マッチ結果がtrueならイベント
	(?(!...)...)			マッチ結果がfalseならイベント
	(?(%...)...)			マッチした内容を動的にマッチパターンに組み込んで・・・説明できない　コンパイラ的なことをしようとした


RegExp_Parser.h				RegExp.hがある程度バグ取りできた時点でマッチパターンのパースを定義済みパーサから組み立てた

RegExp_RTF_parser.h			リッチテキストファイルのパーサー　イベントを拾ってパーシングステートを変更しマッチパターンの変更を行ったりしている

RegExp_Vender.h				マッチ対象のバイトサイズ（Unicode文字列なら２バイト）に合わせたエンジンを作成する
