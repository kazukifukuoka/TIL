# バージョンを指定してyarnでインストール
$ yarn add admin-lte@^3.0

# ファイルが生成される
- node_modules
- package.json
- yarn.lock

# マニフェストファイルの読み込み
- コンソールで確認
$ rails c 
> Rails.application.config.assets.paths #pathを確認

#<Pathname:/Users/kazuki/runteq/runteq/node_modules>

- 読み込み設定  
```
app/assets/javascripts/application.js
//= require rails-ujs
//= require_tree .　→　削除

//= require_tree .は同階層の全てのjsを読み込むので削除して個別に// require hogehogeと記述するのが良い
```  

```
app/assets/javascripts/application.js

//= require admin-lte/dist/js/adminlte.min　追加
```

```
app/assets/stylesheets/admin.scss

@import 'admin-lte/dist/css/adminlte.min'; 追加
```  

# admin.scss、admin.jsの作成
 
# ルーティングの設定
 - namespaceの利用
 namespaceでURLが　admin/　ではじまる。
 
  namespace :admin do
    get 'login', to: 'user_sessions#new'　　(→　 admin_login GET    /admin/login(.:format)     admin/user_sessions#new)
    post 'login', to: 'user_sessions#create'
    get 'starter', to: 'dashboards#index'
  end
    
# コントローラーの設定
layout 'ファイル名'でlayoutを指定できる

controllers/admin/base_controller.rb
```.ruby
class Admin::BaseController < ApplicationController
  before_action :is_admin?

  private
  def is_admin?
    return unless current_user
    return if current_user.admin?
    redirect_to root_path, danger: '権限がありません'
  end
end
```
controllers/admin/user_sessions_controller.rb

```.ruby
class Admin::UserSessionsController < Admin::BaseController

  skip_before_action :require_login
  layout 'admin_login'
  def new

  end

  def create
    @user = login(params[:email], params[:password])
    if @user && @user.admin?
      redirect_to admin_starter_path, success: t('.success')
    else
      redirect_to root_path, danger: '権限がありません'
    end
  end
end
```

controllers/admin/dashboards_controller.rb

```.ruby
class Admin::DashboardsController < Admin::BaseController
  layout 'admin'
  def index
  end

end
```

# ビューファイル作成
 - views/admin/user_session/new
  
node_modules/admin-lte/pages/example/login.html参照（body以下だけで良い。それ以外はlayoutフォルダ）

- views/admin/dashboards/index

node_modules/admin-lte/starter.html参照（body以下だけで良い。それ以外はlayoutフォルダ）

- layout/admin.html.erb

```.ruby

<!DOCTYPE html>
<!--
This is a starter template page. Use this page to start your new project from
scratch. This page gets rid of all links and provides the needed markup only.
-->
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta http-equiv="x-ua-compatible" content="ie=edge">

  <title>AdminLTE 3 | Starter</title>

  <meta name="viewport" content="width=device-width, initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
ーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーー
    <%= stylesheet_link_tag    'admin', media: 'all' %>
    <%= javascript_include_tag 'admin' %>
ーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーー
</head>

  <body>
    <%= yield %>
  </body>
</html>
```
# assetsの設定
config/initialize/assetsに追加

Rails.application.config.assets.precompile += %w( admin.css )
Rails.application.config.assets.precompile += %w( admin.js )

# userにroleカラム を追加
- enumを使用する

t.integer :role, default: 0

user.rb
  enum role: { general: 0, admin: 1 }
  

# 管理者権限付与  
rails cで
@user.role
→　{"general"=>0, "admin"=>1}

@user.admin!
でadminに変わる
