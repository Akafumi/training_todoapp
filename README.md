# training_todoapp

Web系エンジニアへの第一歩、素晴らしい挑戦ですね！

devcontainer（Docker）を使って、「Vue.js（フロントエンド）」、.NET（バックエンド）、「PostgreSQL（データベース）」をすべてコンテナ内で完結させ、簡単なTODOアプリが動くまでの全工程をコードの解説付きで順番に解説します。

1. 全体のディレクトリ構成
まずは、お手元のパソコン（ホスト）の作業フォルダ内に以下の構成を作成してください。まだ backend と frontend の中身は空（またはフォルダ自体未作成）で大丈夫です。

Plaintext
todo-app/
 ┣ .devcontainer/
 ┃  ┣ devcontainer.json
 ┃  ┣ docker-compose.yml
 ┃  ┗ Dockerfile
 ┣ backend/      （後からコマンドで作成）
 ┗ frontend/     （後からコマンドで作成）
2. .devcontainer 内のファイル作成
.devcontainer フォルダの中に、以下の3つのファイルを作成します。

① .devcontainer/Dockerfile
開発コンテナの中に、.NET SDK と Node.js（Vue.js用） を両方インストールするための設定ファイルです。

Dockerfile
# .NET 8.0 の公式SDKイメージをベースにする
FROM mcr.microsoft.com/dotnet/sdk:8.0

# 必要なパッケージのインストール（curlやgitなど）
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Node.js（LTS版）をインストール（Vue.jsのビルド・開発に必要）
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs

# コンテナ内の作業ディレクトリを /workspace に設定
WORKDIR /workspace
② .devcontainer/docker-compose.yml
開発用アプリコンテナ（app）とデータベースコンテナ（db）の2つを連携して起動する設定です。

YAML
version: '3.8'

services:
  # 開発用コンテナ (.NET と Node.js が入る)
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      # ホスト側のプロジェクト全体をコンテナ内の /workspace にマウント
      - ..:/workspace:cached
    # コンテナを常時起動状態にしておくためのコマンド
    command: sleep infinity
    ports:
      - "5000:5000" # .NET API用ポート
      - "5173:5173" # Vue.js (Vite) 用ポート
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    depends_on:
      - db

  # データベース用コンテナ (PostgreSQL)
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: tododb
    ports:
      - "5432:5432"
    volumes:
      # データの永続化用ボリューム
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
③ .devcontainer/devcontainer.json
VS Codeに「どのDocker環境に接続するか」や「入れておくと便利な拡張機能」を教える設定です。

JSON
{
  "name": "Vue & .NET DevContainer",
  // 使うdocker-composeファイルの指定
  "dockerComposeFile": "docker-compose.yml",
  // 接続するサービス名（docker-compose.ymlの app）
  "service": "app",
  // 開いたときのワークスペースのパス
  "workspaceFolder": "/workspace",

  // コンテナ内で有効にしたいVS Code拡張機能
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csharp",    // C# / .NET 開発用
        "Vue.volar",                // Vue.js 開発用 (Vue Official)
        "dbaeumer.vscode-eslint",   // コードの構文チェック
        "esbenp.prettier-vscode"    // コード整形
      ]
    }
  }
}
3. devcontainerの起動方法
VS Codeで todo-app フォルダを開く。

キーボードの Ctrl + Shift + P （Macは Cmd + Shift + P）を押す。

「Dev Containers: Reopen in Container」 を選択してEnterを押す。

初回はビルドに数分かかります。画面左下が Dev Container: Vue & .NET DevContainer に変われば成功です！

4. バックエンド (.NET Web API) の作成
コンテナが起動したら、VS Code内の統合ターミナル（`Ctrl + `` で開けます）を開き、以下のコマンドを上から順に入力します。

Bash
# 1. backendディレクトリに .NET の Web API プロジェクトを作成
dotnet new webapi -o backend --use-controllers

# 2. PostgreSQLと接続するためのライブラリを追加
cd backend
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
💡 ポイント: 今回は簡単のため、Controllerを使ったオーソドックスなWeb API構成にします。

5. フロントエンド (Vue.js + Vite) の作成
続けて、ターミナルでプロジェクトルート（/workspace）に戻り、Vueのプロジェクトを作成します。

Bash
# プロジェクトルートに戻る
cd /workspace

# Viteを使って frontend ディレクトリに Vue (JavaScript) プロジェクトを作成
npm create vite@latest frontend -- --template vue

# 必要なライブラリのインストールと axios（API通信用）の導入
cd frontend
npm install
npm install axios
6. 動作確認のためのワンポイントアドバイス
ここまでで、環境構築とプロジェクトの雛形作成が完了しました！

この後は、それぞれのフォルダで以下の実装を進めていきます。

バックエンド (backend/):

TODOのデータを表すモデル（TodoItem.cs）を作成

PostgreSQLとつなぐためのDBコンテキストを設定 (appsettings.json に接続文字列 Server=db;Database=tododb;Username=postgres;Password=password; を記述)

データの取得・追加を行うコントローラー（TodosController.cs）を作成

フロントエンド (frontend/):

App.vue にTODO一覧の表示フォームと入力欄を作成

axios を使ってバックエンド（http://localhost:5000 など）と通信

まずはここまでエラーなくコンテナが立ち上がり、.NET と Node.js のコマンドが通ることを確認してみてください。途中で分からないコマンドやエラーが出たら、いつでも気軽にご相談くださいね！

環境構築と雛形作成の次のステップへ進みましょう！

次はバックエンド（.NET）のデータベース接続とTODOデータのAPI実装を行います。

以下の手順に沿って、コードを配置・作成していってください。すべてのコードに解説（コメント）を加えています。

ステップ 1: バックエンドの接続設定（PostgreSQL）
まずは、.NET側からPostgreSQLに接続できるように設定ファイルを書き換えます。

1. backend/appsettings.json の編集
backend/appsettings.json を開き、データベースの接続文字列（ConnectionStrings）を追加します。

JSON
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  /* ▼ ここから追加：docker-composeで立てたPostgreSQLへの接続情報 */
  "ConnectionStrings": {
    "DefaultConnection": "Host=db;Database=tododb;Username=postgres;Password=password"
  }
}
解説:

Host=db: docker-compose.yml で定義したデータベースのサービス名（db）を指定しています。devcontainer環境内では、サービス名でコンテナ同士が通信できます。

ステップ 2: データモデル（Entity）の作成
TODOデータの構造（ID、タイトル、完了フラグなど）を定義するクラスを作ります。

1. backend/Models/TodoItem.cs の作成
backend フォルダの中に Models というフォルダを作り、その中に TodoItem.cs ファイルを作成します。

C#
namespace Backend.Models
{
    // TODOデータの設計図となるクラス
    public class TodoItem
    {
        // データの固有ID（主キー）
        public int Id { get; set; }

        // TODOのタイトル（例: 「牛乳を買う」）
        public string Title { get; set; } = string.Empty;

        // 完了しているかどうか（true / false）
        public bool IsCompleted { get; set; } = false;
    }
}
ステップ 3: データベースコンテキストの作成
Entity Framework Coreを使って、C#のコードとPostgreSQLのテーブルを繋ぎます。

1. backend/Data/TodoContext.cs の作成
backend フォルダの中に Data フォルダを作り、その中に TodoContext.cs ファイルを作成します。

C#
using Microsoft.EntityFrameworkCore;
using Backend.Models;

namespace Backend.Data
{
    // データベースとのセッション（やり取り）を管理するクラス
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options) : base(options)
        {
        }

        // TodoItemクラスを元に、データベース内に 'TodoItems' テーブルを作る設定
        public DbSet<TodoItem> TodoItems { get; set; } = null!;
    }
}
ステップ 4: Program.cs の修正（DB接続の登録）
アプリケーションが起動するときに、PostgreSQLへ接続できるように Program.cs を書き換えます。

1. backend/Program.cs の編集
既存の backend/Program.cs を開き、以下のように書き換えます（または必要な部分を追加します）。

C#
using Microsoft.EntityFrameworkCore;
using Backend.Data;

var builder = WebApplication.CreateBuilder(args);

// ▼ 追加：PostgreSQL (Npgsql) を使う設定をコンテナに登録する
builder.Services.AddDbContext<TodoContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

// Controllerを使うためのサービスを追加
builder.Services.AddControllers();

// CORS（フロントエンドからのアクセスを許可する設定）を追加
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// ▼ 追加：作成したCORSポリシーを適用
app.UseCors("AllowAll");

app.UseAuthorization();

app.MapControllers();

app.Run();
解説:

UseNpgsql: 先ほど appsettings.json に書いた接続情報を使ってPostgreSQLに接続するよう指示しています。

AddCors: 別ポート（5173）で動くVue.jsから、バックエンド（5000）へ通信できるようにブロックを解除する設定です。

ステップ 5: APIコントローラーの作成
「TODOリストを取得する」「TODOを追加する」といったリクエストを受け付ける窓口（コントローラー）を作ります。

1. backend/Controllers/TodosController.cs の作成
backend/Controllers フォルダの中に TodosController.cs ファイルを作成します。

C#
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Backend.Data;
using Backend.Models;

namespace Backend.Controllers
{
    // APIのURLのプレフィックスを 「api/todos」 に設定
    Route("api/[controller]")]
    [ApiController]
    public class TodosController : ControllerBase
    {
        private readonly TodoContext _context;

        // 依存性注入（DI）でデータベースのコンテキストを受け取る
        public TodosController(TodoContext context)
        {
            _context = context;
        }

        // 1. 【GET】すべてのTODOを取得する (GET: /api/todos)
        [HttpGet]
        public async Task<ActionResult<IEnumerable<TodoItem>>> GetTodoItems()
        {
            return await _context.TodoItems.ToListAsync();
        }

        // 2. 【POST】新しいTODOを追加する (POST: /api/todos)
        [HttpPost]
        public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
        {
            _context.TodoItems.Add(todoItem);
            await _context.SaveChangesAsync();

            // 作成されたデータと、その固有のURLを返す
            return CreatedAtAction(nameof(GetTodoItems), new { id = todoItem.Id }, todoItem);
        }
    }
}
ステップ 6: データベースのマイグレーション（テーブル作成）
コードの準備ができたので、データベース内に実際のテーブルを作成します。VS Codeの統合ターミナル（backend フォルダに移動した状態）で以下を実行してください。

Bash
# マイグレーションファイルの作成
dotnet ef migrations add InitialCreate

# データベースにテーブルを反映
dotnet ef database update
💡 ポイント: これにより、PostgreSQLコンテナの中に TodoItems テーブルが自動で作られます。

ここまで完了したら、一度バックエンドが正しく動くか確認してみましょう。

ターミナルで dotnet run を実行し、エラーが出ずに起動できればバックエンドの準備は完璧です！

ここまでの実装で、分からない箇所やエラーが出た場合は教えてください。次はフロントエンド（Vue.js）の実装に進みます！

バックエンド（.NET & PostgreSQL）の準備が完了したので、最後のステップであるフロントエンド（Vue.js）の実装と、バックエンドとの連携を進めましょう！

ここでは、シンプルに「TODOの一覧表示」「新しいTODOの追加」「完了/未完了の切り替え」ができる画面を作ります。

ステップ 1: API通信用の設定ファイル作成
フロントエンドからバックエンド（http://localhost:5000）へ簡単にリクエストを送るため、Axiosのインスタンスを作成します。

1. frontend/src/services/api.js の作成
frontend/src の中に services フォルダを作り、その中に api.js ファイルを作成します。

JavaScript
import axios from 'axios';

// バックエンドのURL（開発環境用）をベースURLとして設定
const apiClient = axios.create({
  baseURL: 'http://localhost:5000/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

export default {
  // TODO一覧を取得するAPI呼び出し
  getTodos() {
    return apiClient.get('/todos');
  },
  // 新しいTODOを追加するAPI呼び出し
  createTodo(todo) {
    return apiClient.post('/todos', todo);
  }
};
ステップ 2: 画面コンポーネントの実装
次に、ユーザーが操作する画面（App.vue）を書き換えます。

1. frontend/src/App.vue の書き換え
既存の frontend/src/App.vue をすべて削除し、以下のコードに書き換えてください。すべての処理にコメントを記述しています。

コード スニペット
<script setup>
import { ref, onMounted } from 'vue';
import api from './services/api';

// リアクティブな変数（データの状態管理）
const todos = ref([]);       // TODOリストの配列
const newTitle = ref('');    // 入力フォームのバインド用

// 1. 画面が読み込まれたときにTODO一覧を取得する関数
const fetchTodos = async () => {
  try {
    const response = await api.getTodos();
    todos.value = response.data;
  } catch (error) {
    console.error('TODOの取得に失敗しました:', error);
  }
};

// 2. 新しいTODOを追加する関数
const addTodo = async () => {
  // 空白の場合は何もしない
  if (!newTitle.value.trim()) return;

  try {
    const newTodo = {
      title: newTitle.value,
      isCompleted: false
    };
    // バックエンドに保存を依頼
    const response = await api.createTodo(newTodo);
    
    // 画面側のリストにも追加して即座に反映させる
    todos.value.push(response.data);
    
    // 入力フォームをクリア
    newTitle.value = '';
  } catch (error) {
    console.error('TODOの追加に失敗しました:', error);
  }
};

// コンポーネントがマウントされたら一覧を取得
onMounted(() => {
  fetchTodos();
});
</script>

<template>
  <div class="container">
    <h1>📝 シンプル TODOアプリ</h1>

    <!-- TODO追加フォーム -->
    <form @submit.prevent="addTodo" class="form">
      <input 
        type="text" 
        v-model="newTitle" 
        placeholder="新しいTODOを入力..." 
        class="input"
      />
      <button type="submit" class="btn">追加</button>
    </form>

    <!-- TODO一覧リスト -->
    <ul class="todo-list">
      <li v-for="todo in todos" :key="todo.id" class="todo-item">
        <span :class="{ completed: todo.isCompleted }">
          {{ todo.title }}
        </span>
      </li>
    </ul>
  </div>
</template>

<style scoped>
/* 簡易的なスタイリング */
.container {
  max-width: 500px;
  margin: 40px auto;
  font-family: sans-serif;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
h1 {
  text-align: center;
  color: #333;
}
.form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.btn {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.btn:hover {
  background-color: #35495e;
}
.todo-list {
  list-style: none;
  padding: 0;
}
.todo-item {
  padding: 10px;
  border-bottom: 1px solid #eee;
  font-size: 18px;
}
.completed {
  text-decoration: line-through;
  color: #888;
}
</style>
ステップ 3: アプリを起動して動作確認をしよう！
コードが準備できたら、それぞれのアプリをコンテナ内で起動して動作をテストします。

1. バックエンド（.NET）の起動
VS Codeの統合ターミナルで backend フォルダに移動し、アプリを起動します。

Bash
cd /workspace/backend
dotnet run
成功すると、Now listening on: http://localhost:5000 のようなログが表示されます。

2. フロントエンド（Vue.js）の起動
新しいターミナルタブを開き（VS Codeのターミナル右上の「＋」ボタンなど）、今度は frontend フォルダに移動してViteの開発サーバーを起動します。

Bash
cd /workspace/frontend
npm run dev -- --host
--host をつけることで、コンテナ外やブラウザからアクセスしやすくなります。

http://localhost:5173 で起動した旨が表示されます。

3. ブラウザで確認！
ブラウザを開き、http://localhost:5173 にアクセスしてみてください。

キレイなTODO入力画面が表示されれば成功です！実際に文字を入力して「追加」ボタンを押すと、PostgreSQLデータベースに保存され、画面にTODOが追加されることが確認できます。

ここまで無事に動かせそうか、試してみてくださいね！エラーが出た場合はログの内容を教えていただければ一緒に解決します！


