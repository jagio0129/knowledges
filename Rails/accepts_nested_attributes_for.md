
accepts_nested_attributes_forを定義し、親子関係のある関連モデルで、親モデル作成と同時に子モデルも作成することができます。

```ruby:machine.rb
class Machine < ApplicationRecord
  has_many :parts
  accepts_nested_attributes_for :parts
end
```

```ruby:part.rb
class Part < ApplicationRecord
  belongs_to :machine
end
```

こんな感じで定義しておけば、作成するMachineに紐づくPartモデルを作成できる

```ruby
# 以下のような値をcontrollerで受け取る
params = { machine: { 
           name: '紅蓮可翔式', 
           parts_attributes: [
             { name: '輻射波動砲' }, 
             { name: '飛翔滑走翼' }
           ]
         }}

# newとかでMachineを作成
Machine.create(params[:machine])

Machine.where(name: '紅蓮可翔式').first.parts
=> [id: 1, name: "輻射波動砲弾", machine_id: 1,
    id: 2, name: "飛翔滑走翼", machine_id: 1]
```

これで紅蓮可翔式(Machine)作成時に輻射波動砲弾と飛翔滑走翼(Part)を紐付けて作成できました。
