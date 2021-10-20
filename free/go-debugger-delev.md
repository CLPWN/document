# go debugger [delev](https://github.com/go-delve/delve)の使い方

## インストール
```
$ git clone https://github.com/go-delve/delve
$ cd delve
$ go install github.com/go-delve/delve/cmd/dlv
```

Go version 1.16以上をインストールしている場合にはこちらでもインストール可能
```
$ go install github.com/go-delve/delve/cmd/dlv@latest
```

dlvはどこかビルド済みバイナリが保存されるので、それを探してパスを通して利用する。
```
$ find / -name dlv 2>/dev/null
/home/ensix/go/bin/dlv # 例えばコレ
/home/ensix/go/pkg/mod/github.com/go-delve/delve@v1.7.2/cmd/dlv
```
また、実行にはgccが必要となる。

## 実行ファイル
```
dlv exec exec-file [-- args]
```

## go file
実行例に利用したのはdockerイメージ解析ツールの[dive](https://github.com/wagoodman/dive)

```
$ tree .
.
├── Dockerfile
├── LICENSE
├── Makefile
├── README.md
├── cmd
│   ├── analyze.go
│   ├── build.go
│   ├── ci.go
│   ├── root.go
│   └── version.go
├── dive
│   ├── filetree
│   │   ├── comparer.go
│   │   ├── diff.go
│   │   ├── efficiency.go
│   │   ├── efficiency_test.go
│   │   ├── file_info.go
│   │   ├── file_node.go
│   │   ├── file_node_test.go
│   │   ├── file_tree.go
│   │   ├── file_tree_test.go
│   │   ├── node_data.go
│   │   ├── node_data_test.go
│   │   ├── path_error.go
│   │   └── view_info.go
│   ├── get_image_resolver.go
│   └── image
│       ├── analyzer.go
│       ├── docker
│       │   ├── archive_resolver.go
│       │   ├── build.go
│       │   ├── cli.go
│       │   ├── config.go
│       │   ├── engine_resolver.go
│       │   ├── image_archive.go
│       │   ├── image_archive_analysis_test.go
│       │   ├── layer.go
│       │   ├── manifest.go
│       │   └── testing.go
│       ├── image.go
│       ├── layer.go
│       ├── podman
│       │   ├── build.go
│       │   ├── cli.go
│       │   ├── resolver_linux.go
│       │   └── resolver_notlinux.go
│       └── resolver.go
├── go.mod
├── go.sum
├── main.go
├── runtime
│   ├── ci
│   │   ├── evaluator.go
│   │   ├── evaluator_test.go
│   │   ├── reference_file.go
│   │   └── rule.go
│   ├── event.go
│   ├── export
│   │   ├── export.go
│   │   ├── export_test.go
│   │   ├── file_reference.go
│   │   ├── image.go
│   │   └── layer.go
│   ├── options.go
│   ├── run.go
│   ├── run_test.go
│   └── ui
│       ├── app.go
│       ├── controller.go
│       ├── format
│       │   └── format.go
│       ├── key
│       │   └── binding.go
│       ├── layout
│       │   ├── area.go
│       │   ├── compound
│       │   │   └── layer_details_column.go
│       │   ├── layout.go
│       │   ├── location.go
│       │   ├── manager.go
│       │   └── manager_test.go
│       ├── view
│       │   ├── cursor.go
│       │   ├── debug.go
│       │   ├── details.go
│       │   ├── filetree.go
│       │   ├── filter.go
│       │   ├── layer.go
│       │   ├── layer_change_listener.go
│       │   ├── renderer.go
│       │   ├── status.go
│       │   └── views.go
│       └── viewmodel
│           ├── filetree.go
│           ├── filetree_test.go
│           ├── layer_compare.go
│           ├── layer_selection.go
│           ├── layer_set_state.go
│           └── testdata
│               ├── TestFileShowAggregateChanges.txt
│               ├── TestFileTreeDirCollapse.txt
│               ├── TestFileTreeDirCollapseAll.txt
│               ├── TestFileTreeDirCursorRight.txt
│               ├── TestFileTreeFilterTree.txt
│               ├── TestFileTreeGoCase.txt
│               ├── TestFileTreeHideAddedRemovedModified.txt
│               ├── TestFileTreeHideTypeWithFilter.txt
│               ├── TestFileTreeHideUnmodified.txt
│               ├── TestFileTreeNoAttributes.txt
│               ├── TestFileTreePageDown.txt
│               ├── TestFileTreePageUp.txt
│               ├── TestFileTreeRestrictedHeight.txt
│               └── TestFileTreeSelectLayer.txt
└── utils
    ├── format.go
    └── view.go
```
カレントディレクトリにgo runするファイルがある場合
```
$ pwd
/dive
$ dlv debug
```
ライブラリ等ファイル群のrootディレクトリを指定して使う場合
```
$ dlv debug /dive
```

## dlv実行
先ほど示したdiveのデバッグを例に基本的なコマンドの実行例を示す。
diveはdockerイメージの解析ツールであるため、実行の引数としてコンテナイメージを指定する必要がある。docker pullしておく必要は無いがdockerはインストールされている必要がある。
```
$ pwd 
/dive
$ dlv debug -- alpine
```
### ブレイクポイントの設定
main.goのfunc mainにブレイクポイントを設定する場合
```
(dlv) break main.main
Breakpoint 1 set at 0xab37ea for main.main() ./main.go:33
```
dive/cmd/analyze.goのdoAnalyzeCmd()にブレイクポイントを設定する場合
```
(dlv) break github.com/wagoodman/dive/cmd.doAnalyzeCmd
Breakpoint 2 set at 0xab0172 for github.com/wagoodman/dive/cmd.doAnalyzeCmd() ./cmd/analyze.go:16
```
funcsで利用している関数の一覧が確認できる
```
(dlv) funcs
[diveでは多すぎたので割愛]
```
dive/cmd/analyze.goの70行目にブレイクポイントを設定する場合()
```
(dlv) break ./cmd/analyze.go:70
Breakpoint 2 set at 0xab0856 for github.com/wagoodman/dive/cmd.doAnalyzeCmd() ./cmd/analyze.go:71
```

### 次のブレイクポイントまで実行
continueを使う
```
(dlv) continue
> main.main() ./main.go:33 (hits goroutine(1):1 total:1) (PC: 0xab37ea)
    28:         version   = "No version provided"
    29:         commit    = "No commit provided"
    30:         buildTime = "No build timestamp provided"
    31: )
    32:
=>  33: func main() {
    34:         cmd.SetVersion(&cmd.Version{
    35:                 Version:   version,
    36:                 Commit:    commit,
    37:                 BuildTime: buildTime,
    38:         })
```

### ステップ実行
n or stepを使う。
listで現在の位置を確認してステップ実行を行った。
```
(dlv) list
> main.main() ./main.go:33 (hits goroutine(1):1 total:1) (PC: 0xab37ea)
    28:         version   = "No version provided"
    29:         commit    = "No commit provided"
    30:         buildTime = "No build timestamp provided"
    31: )
    32:
=>  33: func main() {
    34:         cmd.SetVersion(&cmd.Version{
    35:                 Version:   version,
    36:                 Commit:    commit,
    37:                 BuildTime: buildTime,
    38:         })
(dlv) n
> main.main() ./main.go:34 (PC: 0xab37f8)
    29:         commit    = "No commit provided"
    30:         buildTime = "No build timestamp provided"
    31: )
    32:
    33: func main() {
=>  34:         cmd.SetVersion(&cmd.Version{
    35:                 Version:   version,
    36:                 Commit:    commit,
    37:                 BuildTime: buildTime,
    38:         })
    39:
```




### ローカル変数の確認
ローカル変数はlocalsで確認できる。
```
(dlv) break ./cmd/analyze.go:70
Breakpoint 1 set at 0xab090a for github.com/wagoodman/dive/cmd.doAnalyzeCmd() ./cmd/analyze.go:70
(dlv) continue
> github.com/wagoodman/dive/cmd.doAnalyzeCmd() ./cmd/analyze.go:70 (hits goroutine(1):1 total:1) (PC: 0xab090a)
    65:         runtime.Run(runtime.Options{
    66:                 Ci:           isCi,
    67:                 Source:       sourceType,
    68:                 Image:        imageStr,
    69:                 ExportFile:   exportFile,
=>  70:                 CiConfig:     ciConfig,
    71:                 IgnoreErrors: viper.GetBool("ignore-errors") || ignoreErrors,
    72:         })
    73: }
(dlv) locals
userImage = "alpine"
err = error nil
ciConfig = ("*github.com/spf13/viper.Viper")(0xc00011d600)
isCi = false
sourceType = SourceDockerEngine (1)
imageStr = "alpine"
ignoreErrors = false
```

### その他
helpをコピペしておく
```
(dlv) help
The following commands are available:

Running the program:
    call ------------------------ Resumes process, injecting a function call (EXPERIMENTAL!!!)
    continue (alias: c) --------- Run until breakpoint or program termination.
    next (alias: n) ------------- Step over to next source line.
    rebuild --------------------- Rebuild the target executable and restarts it. It does not work if the executable was not built by delve.
    restart (alias: r) ---------- Restart process.
    step (alias: s) ------------- Single step through program.
    step-instruction (alias: si)  Single step a single cpu instruction.
    stepout (alias: so) --------- Step out of the current function.

Manipulating breakpoints:
    break (alias: b) ------- Sets a breakpoint.
    breakpoints (alias: bp)  Print out info for active breakpoints.
    clear ------------------ Deletes breakpoint.
    clearall --------------- Deletes multiple breakpoints.
    condition (alias: cond)  Set breakpoint condition.
    on --------------------- Executes a command when a breakpoint is hit.
    toggle ----------------- Toggles on or off a breakpoint.
    trace (alias: t) ------- Set tracepoint.
    watch ------------------ Set watchpoint.

Viewing program variables and memory:
    args ----------------- Print function arguments.
    display -------------- Print value of an expression every time the program stops.
    examinemem (alias: x)  Examine raw memory at the given address.
    locals --------------- Print local variables.
    print (alias: p) ----- Evaluate an expression.
    regs ----------------- Print contents of CPU registers.
    set ------------------ Changes the value of a variable.
    vars ----------------- Print package variables.
    whatis --------------- Prints type of an expression.

Listing and switching between threads and goroutines:
    goroutine (alias: gr) -- Shows or changes current goroutine
    goroutines (alias: grs)  List program goroutines.
    thread (alias: tr) ----- Switch to the specified thread.
    threads ---------------- Print out info for every traced thread.

Viewing the call stack and selecting frames:
    deferred --------- Executes command in the context of a deferred call.
    down ------------- Move the current frame down.
    frame ------------ Set the current frame, or execute command on a different frame.
    stack (alias: bt)  Print stack trace.
    up --------------- Move the current frame up.

Other commands:
    config --------------------- Changes configuration parameters.
    disassemble (alias: disass)  Disassembler.
    dump ----------------------- Creates a core dump from the current process state
    edit (alias: ed) ----------- Open where you are in $DELVE_EDITOR or $EDITOR
    exit (alias: quit | q) ----- Exit the debugger.
    funcs ---------------------- Print list of functions.
    help (alias: h) ------------ Prints the help message.
    libraries ------------------ List loaded dynamic libraries
    list (alias: ls | l) ------- Show source code.
    source --------------------- Executes a file containing a list of delve commands
    sources -------------------- Print list of source files.
    types ---------------------- Print list of types

Type help followed by a command for full documentation.
```