
- 原因
  - Assembly-CSharp.dll（.NET 3.5)が、なぜか.NET 4のライブラリを参照してしまっているのが原因
- 回避手順
  1. Assembly-CSharp.dll (必要なら Assembly-CSharp-fastpass.dllも) を別ディレクトリにコピー
  2. VisualStudio のプロジェクトの参照では、コピーした先の .dll を参照
- 効果
  - Assembly-CSharp.dll が間接的に参照している、 .NET 4.0 が必要な dll まで読み込みに行かなくなり、.NET 3.5でもコンパイル・エラーにならなくなる
