---
title: stubとmockについて
tags: ["rspec"]
excerpt: stub, mockに関するまとめ
---
# stub/mockとは
- 単体テスト(Unit Test)で必要となるパーツを擬似的に再現するための仕組み。
- 擬似的にパーツを再現する必要性は以下
  1. 全てを「本物」でテストしようとすると、全てが揃わないとテストできない可能性がある
  1. 例えば時刻を表すオブジェクトのように状況によって変化するオブジェクトがテストを煩雑にする

## stubの目的
- 本当にテストしたいところだけをテストできるように、本筋に関係ない部分(バリデーションなど)をとにかく動くようにする、動かしたい場所まで辿り着くこと。
  - 例えて言うなら、ペンキを塗ったりエアブラシを使うときに、塗料が付いて欲しくない部分に貼るマスキングテープをイメージするとよい。テストしたい箇所以外はスタブで塞いでしまうわけ。

```ruby
#vtypes_controller_spec
describe "POST create" do
  it "create_from_text が呼ばれ，client_vtypes_path を再表示すること" do
    Vtype.stub(:type_string).and_return(10)      # ここがスタブ
    post :create, text:"data_text", vtype:"10"   # テストしたいアクション
    response.should redirect_to(client_vtypes_path(filtered_vtype:"10")) # アクションの結果をテスト
  end
end
```
4行目では、Vtypeモデルtype_stringメソッドを偽って、type_stringが10という値を返したということにしてくれている。

```ruby
# name:: 種別 - 日本語 (string)，null 不可
# sort_order:: 並び順カラム (integer)，null 不可
class Vtype < ActiveRecord::Base
  def self.type_string(num)
    return self.find_by_sort_order(num).name
  end
end
```

なお以下はコントローラのcreateアクションの抜粋
```ruby
#vtypes_controller
def create
  @vtype  = params[:filtered_vtype]
  if @vtype.save
    flash[:notice] = "#{@v.name}を作成しました"
    redirect_to client_vtypes_path(filtered_vtype: @vtype) and return
  else
    @vtype = params[:filtered_vtype]
    render action:'new'
  end
end
```
- スタブのメリット
  - オブジェクトやメソッドのふりをするだけなので、データを用意するfixtureやfactoryよりも高速である点。
  - 本筋に関係ない部分が未実装であっても、スタブを使用できるので完全にシステムをだますことができる。
- デメリット
  - スタブで塞いだ箇所はテストされたことにならない
  - スタブで塞いだ部分は結合テストなど他のどこかの箇所で最終的にテストされていなければならない。

## mockの目的
- モックはスタブと似た面があるが、モックはそれ自体がテストになり得る点がスタブとは決定的に異なる
  - テストする対象のオブジェクトが実装されてないとモックは使えない。

```ruby
describe "POST create" do
  before :each do
    mock_budget.should_receive(:budget_main).and_return(mock_budget_main)
    mock_budget.should_receive(:budget_owner).and_return(mock_budget_owner)
    mock_budget.should_receive(:budget_unit).and_return(mock_budget_unit)
  end

  it "redirects to new_contract_path when creation of contract succeeds" do
    mock_budget.should_receive(:save).and_return(true)
    post :create, budget:{these: :params}
    response.should redirect_to(new_contract_path(budget_id:mock_budget.id))
  end
end
```

# モックとスタブの使い分け
- mock
  - 「オブジェクトのメソッドがどう呼ばれて何を返すか」というインタフェースも含めたテストのために使う
- sub
  - テストをスムーズに行うために「あるオブジェクトのメソッドが呼ばれたら、ある戻り値を返す」ために使う

# 参考サイト
- [[RSpec]モックとスタブの使い方](https://techracho.bpsinc.jp/hachi8833/2014_07_10/17971)
