# SECCON CTF 2020 Pwn - lazynote

問題ファイル: https://bitbucket.org/ptr-yudai/writeups-2020/src/master/SECCON_2020_Online_CTF/lazynote/files/

参考: https://faraz.faith/2020-10-13-FSOP-lazynote/

---

## 問題の詳細

1. 実行すると下記のようなメニューを表示し、4つの選択肢を与える。
1. 選択肢1は、allocation_sizeread_size、およびdataを設定する。
1.1. ここで割り当てと読み取りサイズに、0または負の数を入力できないことを確認。
1. callocを使用してallocation_sizeバイトを割り当てる。この割り当てられたメモリ領域をbufと呼ぶ。
1. 次に、read_size <= allocation_sizeかどうかを確認し、その場合は allocation_size = read_sizeを設定。(ここの処理に脆弱性はない)
1. 次に、allocation_sizeバイトをbufに読み取る。
1. そして、buf[read_size - 1] = 0 を設定。これにより、read_size > allocation_size の場合、NULL バイトの境界外書き込みが発生する。

上記の操作を4回繰り返すとプログラムは終了する。

```
$ ./chall
👶 < Hi.
1.🧾 / 2.✏️ / 3.🗑️ / 4.👀
> 1
alloc size: 5
read size: 5
data: AA
1.🧾 / 2.✏️ / 3.🗑️ / 4.👀
> 
```

## LIBCソースコードより手がかりを探す
```
// libio/bits/libio.h:245

struct _IO_FILE {
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;	/* Current read pointer */
  char* _IO_read_end;	/* End of get area. */
  char* _IO_read_base;	/* Start of putback+get area. */
  char* _IO_write_base;	/* Start of put area. */
  char* _IO_write_ptr;	/* Current put pointer. */
  char* _IO_write_end;	/* End of put area. */
  char* _IO_buf_base;	/* Start of reserve area. */
  char* _IO_buf_end;	/* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
#if 0
  int _blksize;
#else
  int _flags2;
#endif
  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

#define __HAVE_COLUMN /* temporary */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  /*  char* _save_gptr;  char* _save_egptr; */

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```

stdoutはバッファなし、ラインバッファリング、または完全バッファリングができる。バッファリングモードに応じて、プログラムは異なる出力を行う。

- バッファなし: プログラムはできるだけ早く任意の文字をstdoutする。
- ラインバッファリング: プログラムは改行を見ると文字を出力する。
- 完全バッファリング: プログラムは、stdoutのバッファがいっぱいになると文字を出力する。
