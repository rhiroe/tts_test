# TTS技術検証

Rails 5.2  
Ruby 2.5  
gem 'tts'  
`brew install mpg123` ファイルを保存せず、直接再生する場合必要

## お試し
ユーザー登録  
`http://localhost:3000/users/new`  
再生
`http://localhost:3000/`

## 再生
`'こんにちは'.play 'ja'`

## 保存
`'こんにちは'.to_file 'ja'`

# 定期実行
１分ごとに`:count`の少ない順にユーザー名が２人ずつ掃除当番に呼ばれる

`gem 'whenever', require: false`  
`$ wheneverize .`で`config/schedule.rb`を作成

## 注意点
ActiveRecordをいじるため以下が必要  
[`=> :environment do`](https://github.com/eRy-sk/tts_test/blob/master/lib/tasks/speak.rake#L3)  
それを環境ごとに定期実行で正常に動かすために設定が必要  
[`set :environment, rails_env`](https://github.com/eRy-sk/tts_test/blob/master/config/schedule.rb#L10)

[playメソッド](https://github.com/c2h2/tts/blob/master/lib/tts.rb#L82)
```ruby
fn = "tts_playonce"
self.to_file(lang, fn) # 一時ファイルの作成
times.times{|i| `mpg123 -q #{fn}`}　# 引数timeの処理（再生回数）
File.delete(fn) # 一時ファイルの削除
```

# 音声の確認
ルーティング
```routes.rb
resources :users do
  member do
    get 'sound_for'
  end
end
```
コントローラ
```users_controller.rb
def sound_for
  instant_file = Tempfile.open(['instant_file', '.mp3'])
  @user.name.to_file('ja', instant_file.path)
  send_file(instant_file)
end
```
ビュー
```show.html.erb
<%= audio_tag sound_for_user_path(@user), controls: true %>
```
