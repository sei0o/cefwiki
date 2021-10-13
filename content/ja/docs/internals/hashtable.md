---
title: "ハッシュテーブル"
linkTitle: "ハッシュテーブル"
weight: 8
date: 2017-01-04
description: ''
---

FaceやFIBの管理には独自実装のハッシュテーブル（Key-Value store）が使われています。`cef_lhash` 関連は[リストハッシュ](/docs/internals/listhash)を参照してください。

## 操作のための関数

```c:cef_hash.h
int
cef_hash_tbl_item_num_get (
	CefT_Hash_Handle handle
);
int
cef_hash_tbl_def_max_get (
	CefT_Hash_Handle handle
);
int
cef_hash_tbl_item_max_idx_get (
	CefT_Hash_Handle handle
);
void*
cef_hash_tbl_elem_get (
	CefT_Hash_Handle handle,
	uint32_t* index
);
void*
cef_hash_tbl_no_col_item_get (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);

void*
cef_hash_tbl_item_check (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);
int
cef_hash_tbl_item_check_exact (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);
void* 
cef_hash_tbl_item_set_prg (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen,
	void* elem
);
void* 
cef_hash_tbl_item_get_prg (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);

```

### `CefT_Hash_Handle cef_hash_tbl_create(uint32_t table_size)`
`table_size`個のレコードを持つハッシュテーブルを作成する。作成時に全体のメモリをごっそり確保（`malloc`）するので、csmgrdが4GB固定で必要なのもたぶんそれが原因。

### `void cef_hash_tbl_destroy(CefT_Hash_Handle handle)`
`handle`が指し示すテーブルを削除する。

### `void* cef_hash_tbl_item_get (CefT_Hash_Handle handle, const unsigned char* key, uint32_t klen);`
テーブルの`key`に対応する`elem`（値へのポインタ）を返す。見つからなければ`NULL`を返す。

### `void* cef_hash_tbl_item_get_for_app (CefT_Hash_Handle handle, const unsigned char* key, uint32_t klen);`
テーブルの`key`について前方一致検索を行う。テーブルのあるレコードが、前方一致検索の対象であれば、
	
- `key`がそのレコードのキーと一致する
- あるいは、引数の`key`の長さが、そのレコードのキーの長さ + 5バイト以上であり、引数の`key`の文字列が`NULL`で終わり、その１バイト後に`0x01`がある
	- TODO: どういうことかわからん

場合にそのレコードの`elem`を返す。前方一致検索の対象でない場合は単に`key`が一致するか見る。

```c
            /* eg) ccn:/test, ccn:/test/a */
            /*                         ^^ */
            /* separator(4) and prefix(more than 1) */
```

バグ？: 通常の`item_get()`では最後までくると先頭から探索をやり直すが、この関数は最後まで来たらそこで検索を終了する。

### `int cef_hash_tbl_item_set(CefT_Hash_Handle handle, const unsigned char *key, uint32_t klen, void *elem)`
`handle`が指し示すテーブルに、`key`（キー）と`elem`（値へのポインタ）のペアでレコードを作って格納する。すでに`key`に対応する値があれば、上書きされる。成功すればハッシュテーブル中の位置（`ht->tbl`中の添字）を返し、失敗すれば`CefC_Hash_Faile`を返す。

`elem`が値へのポインタであり、テーブル自身は値を持たないことに注意されたい。

### `int cef_hash_tbl_item_set_for_app(CefT_Hash_Handle handle, const unsigned char *key, uint32_t klen, uint8_t opt, void *elem)`
`cef_hash_tbl_item_set()`と同じだが、`opt`を引数にとる。これを1 (true)に設定すると、そのレコードは前方一致検索の対象になる。`cef_hash_tbl_item_get_for_app()`を参照。

### `void *cef_hash_tbl_item_remove(CefT_Hash_Handle handle, const unsigned char *key, uint32_t klen)`
`handle`が指し示すテーブルの、`key`に対応するレコードを削除する。

### `void *cef_hash_tbl_item_get_from_index(CefT_Hash_Handle handle, uint32_t index)`
`handle`が指し示すテーブルの、`index`番目にあるレコードを取り出し、その値へのポインタを返す。削除済みや未書き込みのところは`NULL`を返す。

### `void *cef_hash_tbl_item_check_from_index(CefT_Hash_Handle handle, uint32_t *index)` 
`handle`が指し示すテーブルの、`index`番目にあるレコードを取り出し、その値へのポインタを返す。また、ポインタで渡した`index`の値をその位置にする。レコードが見つからなかった場合は`NULL`を返し、`index`の値を0にする。

### `void *cef_hash_tbl_item_remove_from_index(CefT_Hash_Handle handle, uint32_t index)` 
`handle`が指し示すテーブルの、`index`番目にあるレコードを削除して、その値へのポインタを返す。テーブル自身が値を持っているわけではないので、削除後にポインタを使っても問題はない。

### `uint32_t cef_hash_tbl_hashv_get (CefT_Hash_Handle handle, const unsigned char* key, uint32_t klen);`
`key`に対応するハッシュ値を返す。同じキーであっても、テーブルによって生成されるハッシュ値は異なる（テーブルが内部にseedを持つ）。

## 構造

`CefT_Hash`

### ハッシュの計算
`cef_hash_number_create()`関数を参照。keyのMD5の12バイト目から4バイト（32bit）とってハッシュにする。

### 書き込み
`cef_hash_tbl_item_set()`関数。
keyからハッシュをつくって、`index = hash %  ht->elem_max(const)` とする。
1. indexにあるレコードが空（klen == 0）なら、そこに格納する
2. 空でないなら、そこから一つずつ歩いていって、空のレコード（またはhash, keyが同じレコード）が見つかったらそこに格納する
	1. ここで、削除済（klen == -1）のレコードが見つかったらそのうちの一つだけ場所をメモしておく
3. 最後（elem_max）まで歩いてもまだ見つからなかったら、先頭に戻ってまた歩く
	1. 削除済みも同様にメモしておく

keyが同じレコードがあるなら、レコードを新規作成するのではなくそれを書き換えるべき。よって削除済みで空になったレコードを見つけても、即座に入れることはしないでまずは探すということ。ただし、一度もデータが入ったことのない場所（klen == 0）より後ろに目的のレコードがあることはないので、klen == 0を見つけると新規のkeyであることが確定して書き込める（klen == 0 なレコードを飛ばして歩いていくことはない）。

例：

|index|0|1|2|3|4|5|
|-----|-|-|-|-|-|-|
|key|hogehoge|piyopiyo|hugahuga|foobar|NULL|NULL|
|klen|8|8|-1|6|0|0|
|elem|0xdeadbeef|0xdeadbeef|0xdeafbeef|NULL|NULL|NULL|

このテーブルで、2番目のkey `hugahuga` のレコードは削除済み。4番目と5番目のレコードが未書き込み。ここで、
- key `AAA`（ハッシュ値0）に値をセットしようとすると、0, 1, 2…と歩いて、書き込み先は2番目になる。
- key `foobar`（ハッシュ値1）に値をセットしようとすると、1, 2...と歩いて、途中で削除済みの2番目を発見するが一旦保留する。その次の3番目に同じkeyを発見するので、そこに書き込まれる。

### 読み込み
`cef_hash_tbl_item_get()`関数。ハッシュ値に対応する位置から未書込が出るか、目的のkeyを見つけるまで歩く。