# fullcalendar

## 公式リファレンス

- [fullCalendar](http://fullcalendar.io/)
- [gem 'fullCalendar-rails'](https://github.com/bokmann/fullcalendar-rails)

## 実行環境

- CentOS6.5
- Rails v4.2.2
- Ruby v2.3.1p112
- gem v2.6.6
- fullcalendar v2.8.0
- jquery-rails v4.1.1

## 使い方

1. **gemを追加する。**

  ```ruby
  gem 'fullcalendar-rails'
  gem 'momentjs-rails'

  # Use jquery as the JavaScript library
  gem 'jquery-rails'
  ```

  追加したら`$ bundle install`

2. **カレンダーの表示**

  controllerとviewを作成

  ```
  $ bundle exec rails g controller calender index`
  ```

  viewに表示領域を作成

  ```html
  [app/views/calendar.index.html.erb]

  <div id="calendar"></div>
  ```

  calendar.jsの中でfullcalendarを呼ぶ。coffeescriptはめんどいのでjsに変更しても可

  ```javascript
  [app/assets/javascript/calendar.js]

  $(document).ready(function() {
  $('#calendar').fullCalendar({
  })
  });
  ```

  app/assets/javascript/application.jsのrequire_tree .の前に以下の二行を追加

  ```javascript
  //= require moment
  //= require fullcalendar
  ```

  app/assets/stylesheets/application.cssのrequire_tree .の前に以下の一行を追加

  ```css
  *= require fullcalendar
  ```

  `$ rails s`を実行し、<http://localhost:3000/calendar/index>にアクセスすると表示

## オプション

カレンダー全体の設定をする。以下例

```javascript
$('#calendar').fullCalendar({
    //ヘッダーの設定
    header: {
        //それぞれの位置に設置するボタンやタイトルをスペース区切りで指定できます。指定しない場合、非表示にできます。
        // 'title'→月・週・日のそれぞれの表示に応じたタイトル
        // 'prev'→前へボタン
        // 'next'→次へボタン
        // 'today'→当日表示ボタン
        left: 'today prev', //左側に配置する要素
        center: 'title', //中央に配置する要素
        right: 'next' //右側に配置する要素
    },

    height: 960, //高さをピクセルで指定
    defaultView: 'agendaDay', //初めの表示内容を指定　内容はこちらを参照→ http://fullcalendar.io/docs/views/Available_Views/
    aditable: true, //trueでスケジュールを編集可能にする
    allDaySlot: false,　//falseでagendaDay表示のときに全日の予定欄を非表示にする

    // 月や曜日を日本語に変更
    monthNames: ['１月','２月','３月','４月','５月','６月','７月','８月','９月','１０月','１１月','１２月'],
      monthNamesShort: ['１月','２月','３月','４月','５月','６月','７月','８月','９月','１０月','１１月','１２月'],
      dayNames: ['日曜日','月曜日','火曜日','水曜日','木曜日','金曜日','土曜日'],
      dayNamesShort: ['日','月','火','水','木','金','土'],

    //時間の表示フォーマットを指定する　指定方法はこちらを参照→http://momentjs.com/docs/#/displaying/format/
    timeFormat: {
        agenda: 'h(:mm)'
    },
    slotEventOverlap: false, //スケジュールが重なったとき、重ねて表示するかどうか（falseにすると、重ねずに表示する）
    axisFormat: 'H:mm', //時間軸に表示する時間の表示フォーマットを指定する(ヒョジ方法はtimeFormatと同じ)
    slotDuration: '01:00:00', //表示する時間軸の細かさ
    snapDuration: '01:00:00', //スケジュールをスナップするときの動かせる細かさ
    minTime: "00:00:00", //スケジュールの開始時間
    maxTime: "24:00:00", //スケジュールの最終時間
    defaultTimedEventDuration: '01:00:00', //画面上に表示する初めの時間(スクロールされている場所)

    // イベントをクリックしたときに実行
    eventClick: function(event) {
      /* 引数
      ・event(イベントの開始終了日時なの情報)
      ・allDay(全日をクリックしたかどうか全日の場合はtrue)
      ・jsEvent(座標などjsの情報)
       */
    },

    //イベントじゃないところをクリックしたとき(日をクリックしたとき)に実行
    dayClick: function(date){
      /* 引数
      ・date(slotの依存する日付)
      ・allDay(全日をクリックしたかどうか全日の場合はtrue)
      ・jsEvent(座標などjsの情報)
      ・view(カレンダータイトルや表示モードの情報)
       */
    },

    // 登録してある予定(イベント)の上をマウスが通過した際の処理を記述できます
    eventMouseover: function(){
      /*引数
      ・event(イベントの開始終了日時なの情報)
      ・allDay(全日をクリックしたかどうか全日の場合はtrue)
      ・jsEvent(座標などjsの情報)
      */
    },

    // 登録してある予定(イベント)の上からマウスが離れた際の処理を記述できます
    eventMouseout: function(){
      /* 引数
      ・event(イベントの開始終了日時なの情報)
      ・allDay(全日をクリックしたかどうか全日の場合はtrue)
      ・jsEvent(座標などjsの情報)
      */
    },

    // 予定表のslotをクリックした際の処理を記述できます
    select: function(){
      /*引数
      ・startDate(slotの開始日付)
      ・endDate(slotの終了日付)
      ・allDay(全日をクリックしたかどうか全日の場合はtrue)
      ・jsEvent(座標などjsの情報)
      ・view(カレンダータイトルや表示モードの情報)
       */
    },

    // イベント(予定)毎の描画時の処理を記述できます
    eventRender: function(){
      /* 引数
      ・event(イベントの開始終了日時なの情報)
      ・element(イベントのhtml要素情報)
      */
    },

    // カレンダー表記時(ページング等)毎の描画時の処理を記述できます
    viewRender : function(){
      /*
      ・view(カレンダータイトルや表示モードの情報)
      ・element(html要素情報)
       */
    },

    droppable: true, //外部要素からのドラッグアンドドロップを可にする
    drop: function(date){ //外部要素からドラッグアンドドロップしたときに実行

    },
    eventDragStop: { //カレンダー上にドラッグし終わったときに実行

    },

    // イベントをドラッグ＆ドロップ（予定の変更）後に呼ばれる
    eventDrop: function(event, delta, revertFunc, jsEvent, ui, view) {
    },

    //カレンダーを再描画
    $('#calendar').fullCalendar('rendar');

    //カレンダーを削除
    $('#calendar').fullCalendar('destroy');

    //イベントを追加
    $('#calendar').fullCalendar('renderEvent', event, true); //eventはeventオブジェクト

    //イベントを更新
    $('#calendar').fullCalendar('updateEvent', event);

});
```

## Eventsオブジェクト

カレンダーに表示するデータはEventsオブジェクトで渡す。Eventsオブジェクトの主なプロパティは以下。

- **title** タイトル

- **start** 開始日時

- **end** 終了日時

- **testcolor** 文字色

- **color** 背景色

- **url** イベントにリンクを貼る際に利用

## イベントを登録する

[こちら](http://qiita.com/siguremon/items/10216d15471dfe5ba572)を参考に、イベントをカレンダーから登録できるようにする。

はじめにscaffoldでeventを生成。

```
bundle exec rails g scaffold event title start:datetime end:datetime color allDay:boolean
bundle exec rake db:migrate
```

calendar.jsの最終的な追加分は以下。

```javascript
$(document).ready(function() {
    var select = function(start, end, allDay) {
        var title = window.prompt("title");
        var data = {event: {title: title,
                            start: start.format(),
                            end: end.format(),
                            allDay: allDay}};
        $.ajax({
            type: "POST",
            url: "/events",
            data: data,
            success: function() {
                calendar.fullCalendar('refetchEvents');
            }
        });
        calendar.fullCalendar('unselect');
    };

    var calendar = $('#calendar').fullCalendar({
        events: '/events.json',
        selectable: true,
        selectHelper: true,
        ignoreTimezone: false,
        select: select
    });
});
```

startとendにそれぞれ.format()をしないと`Uncaught TypeError: Cannot read property '_calendar' of undefined`となり、正しく登録されなかったので注意。

## カレンダー上の変更に応じたイベント更新

もともと作成されていたイベントを違う日にそのまま移動させたい時に、D&Dで移動させる。移動先にもとのイベントをコピーし、移動元を削除する流れ。eventDropにD&D後の処理を記述する。

変更にはjquery-uiが必要なので`gem 'jquery-ui-rails'`を追加。

application.jsに//= require jquery-ui、application.cssに*= require jquery-uiを追加する。

fullcalendarのオプションに`ediatable: true`を追加。

```javascript
var updateEvent = function(event, revertFunc) {
    var title = window.prompt("Edit title", event.title);
    var url = "/events/" + event.id;
    var data = {_method: 'PUT',
            event: {title: title,
                start: event.start,
                end: event.end,
                allDay: event.allDay}};
    $.ajax({
        type: "POST",
        url: url,
        data: data,
        success: function() {
        calendar.fullCalendar("refetchEvents");
        },
        error: revertFunc
    });
};

var calendar = $("#calendar").fullCalendar({
    events: "/events.json",
    selectable: true,
    selectHelper: true,
    ignoreTimezone: false,
    editable: true,
    select: select,
    eventClick: updateEvent,
    eventResize: updateEvent,
    eventDrop: updateEvent
});
```

## その他

**・ todayの背景色を変更**

```css
[calendar.scss]

.fc-day.fc-today {
    color: #fff !important;
    background-color: yellow;
}
```

**・ todayに印をつける**

```css
[calendar.scss]

.fc-day.fc-today {
    position: relative;
}

.fc-day.fc-today::before,
.fc-day.fc-today::after {
    content: '';
    position: absolute;
    top: 0;
    right: 0;
    border-color: transparent;
    border-style: solid;
}

.fc-day.fc-today::before {
    border-width: 1.5em;
}

.fc-day.fc-today::after {
    border-radius: 0;
    border-width: 1.5em;
    border-right-color: red;
    border-top-color: red;
}
```

## 参考サイト

<http://qiita.com/siguremon/items/73651af19babd22fe012> <http://sterfield.co.jp/designer/%E5%A4%9A%E6%A9%9F%E8%83%BD%E3%81%AE%E3%82%AB%E3%83%AC%E3%83%B3%E3%83%80%E3%83%BC%E3%82%92%E5%AE%9F%E8%A3%85%E3%81%99%E3%82%8Bjquery%E3%83%97%E3%83%A9%E3%82%B0%E3%82%A4%E3%83%B3%E3%81%AEfullcalendar/> <http://shimz.me/blog/jquery/1265>
