---
title: "リストハッシュ"
linkTitle: "リストハッシュ"
weight: 8
date: 2017-01-04
description: ''
---

csmgrで使われているリストハッシュと`cef_lhash_*`系の関数についての解説。

TODO: リストハッシュなんてものは存在しない？ハッシュテーブルの一種？

## 操作のための関数

```c:cef_hash.h

CefT_Hash_Handle
cef_lhash_tbl_create (
	uint32_t table_size
);

CefT_Hash_Handle
cef_lhash_tbl_create_u32 (
	uint32_t table_size
);

void
cef_lhash_tbl_destroy (
	CefT_Hash_Handle handle
);
int
cef_lhash_tbl_item_set (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen,
	void* elem
);
void*
cef_lhash_tbl_item_get (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);
void*
cef_lhash_tbl_item_remove (
	CefT_Hash_Handle handle,
	const unsigned char* key,
	uint32_t klen
);
```

## 内部構造
ハッシュの計算はハッシュテーブルと同じ。

`ht->tbl` がまずelem_maxの配列になっている。ここまではハッシュテーブルと同じ。ハッシュテーブルの場合はハッシュが衝突すると後ろにずれていったのだが、ここでは配列中の位置はそのままデータをねじ込む。ねじ込めるように、各要素（`CefT_List_Hash_Cell`）はnextというプロパティを持っていて、これにポインタを継いでいくことで連結リストができる。実装では、追加したい要素を連結リストの先頭に置いて、もとの連結リストを後に連ねるようにしている。

なお、ハッシュテーブルの要素（`CefT_Hash_Table`）ではkeyの長さは固定長（`CefC_Max_KLen`）だったが、リストハッシュではkeyの長さに応じてメモリを確保している。つまり、要素を作るときに、`CefT_List_Hash_Cell + klen`のようにまとめて確保して連結リストにつなげている。

TODO: 例