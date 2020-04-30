### cocoonとは
- 動的に入力フォームを増減できるようになるgem

### 導入方法（Rails 6の場合、jqueryを導入済み）
`gem "cocoon"`  
`yarn add github:nathanvda/cocoon#c24ba53`  

- モデル
親
`has_many :children, dependent: :destroy`  
`accepts_nested_attributes_for :children, allow: true`  

子
`belongs_to :parent`  

- コントローラー
```.ruby
def new
    @parent = Parent.new
    @children = @parent.children.build
    @grandchildren = @children.grandchildren.build
  end

  def create
    @parent = Parent.new(parent_params)
    @parent.save
  end

private
def parent_params
      params.require(:parent).permit(:p_name, children_attributes: [:id, :c_name, :_destroy, grandchildren_attributes: [:id, :g_name, :_destroy]])
end
```

- ビュー

parent.html.erb

```.ruby
<%= form_with(model: @parent, local: true) do |f| %>
   略
   
        <%= f.fields_for :children do |f| %>
            <%= render "children_fields", f: f %>
        <% end %>
        <div class="links">
            <%= link_to_add_association f, :children %>
        </div>
    略
<% end %>
```
_children_fields.html.erb

```.ruby
<div class="nested-fields">
    <%= f.label "name" %>
    <%= f.text_field :name %>
    
    <%= link_to_add_association f %>
</div>
```

- 注意
 
  フォームは部分テンプレートで実装する
ファイル名に規約があるらしく、_モデル名_fieldsにする必要がある

  
部分テンプレート先では
全体をnested-fieldsクラスで囲む必要がある
「削除」のところはリンクになっていて、部分テンプレート側に記述
