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
