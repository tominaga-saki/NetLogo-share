turtles-own [
  destination
  enter-time
  entered-route?
  uses-escalator?
  route-xcor
  route-pcolor
]

globals [
  total-time
  exit-count
  route-entry-ycor
  route-goal-ycor
  ;; エスカレータを利用する割合（0～1）
  escalator-ratio
  ;; setup直後に発生させる人数
  initial-passenger-count
  ;; 電車到着時の人数（最小人数 + 0～追加人数未満）
  train-passenger-min
  train-passenger-random
  ;; 人同士を避けるために検出する範囲（グリッド単位）
  person-range
]

to setup


  ;; Displayに収まるようにWorldの表示サイズを調整
  resize-world -20 20 -30 30
  set-patch-size 10

  clear-all

  ask patches [
    set pcolor white
  ]


  ;; Left and right walls
  ask patches with [pxcor = -13 or pxcor = 10][
    
  
    set pcolor black
  ]

  ;; Upper and lower walls
  ask patches with [pycor = 30 or pycor = -30][
    set pcolor black
  ]
  
   ;;ホームドア1
  ask patches with [
    pxcor = -13 and
    pycor >= -25 and
    pycor <= -19

  ] [
    set pcolor yellow
  ]
  
  ;;ホームドア2
 
     ask patches with [
       pxcor = -13 and
       pycor >= -9 and
    pycor <= -3
   ] [
    set pcolor yellow
  ]
 
  ;;ホームドア3
 
     ask patches with [
       pxcor = -13 and
       pycor >= 7 and
       pycor <= 13 
   ] [
    set pcolor yellow
  ]
  
  ;;ホームドア4
 
     ask patches with [
       pxcor = -13 and
       pycor >= 23 and
       pycor <= 29 
   ] [
    set pcolor yellow
  ]
  
 
  ;; Stairs 下り
  ask patches with [
    pxcor > -10 and
    pxcor <= -7 and
    pycor >= -12 and
    pycor <= 12
  ] [
    set pcolor gray
  ]
  ;; Stairs 上り
  ask patches with [
    pxcor > -7 and
    pxcor <= 0 and
    pycor >= -12 and
    pycor <= 12
  ] [
    set pcolor brown
  ]

  ;; Escalator（階段の右側）
  ask patches with [
    pxcor > 0 and
    pxcor <= 3 and
    pycor >= -12 and
    pycor <= 12
  ] [
    set pcolor turquoise
  ]
  
  ;; Escalator（階段の右側）
  ask patches with [
    pxcor > 3 and
    pxcor <= 6 and
    pycor >= -12 and
    pycor <= 12
  ] [
    set pcolor green
  ]
  
  set total-time 0
  set exit-count 0
  set route-entry-ycor min [pycor] of patches with [pcolor = brown]
  ;; 階段・エスカレータの上端から1グリッド先をゴールにする
  set route-goal-ycor (max [pycor] of patches with [pcolor = brown]) + 1
  ;; 0.0 = 全員が階段、0.5 = 半数、1.0 = 全員がエスカレータ
  set escalator-ratio 0.5
  ;; 開始時の人数
  set initial-passenger-count 30
  ;; 電車到着時の最低人数
  set train-passenger-min 20
  ;; 追加されるランダム人数  0にすると毎回固定人数
  set train-passenger-random 10
  ;; 0.5 = 半グリッド分、1.0 = 1グリッド分
  set person-range 0.5

  reset-ticks
  
end

to go

  if ticks = 0 [
    create-passengers initial-passenger-count
  ]
  
  ask turtles [

    ;; 選択した階段またはエスカレータに入ったら、その出口へ向かう
    if not entered-route? and [pcolor] of patch-here = route-pcolor [
      set entered-route? true
      set destination patch route-xcor route-goal-ycor
    ]

    ;; 選択した経路を通過し、その先へ到達したら記録して消滅する
    if entered-route? and ycor >= route-goal-ycor [
      let travel-time ticks - enter-time

      set total-time total-time + travel-time
      set exit-count exit-count + 1
      die
    ]

    ;; 1. 現在の目的地を向く
    face (destination)

    ;; 2. 目の前（距離1以内）に壁がある場合は立ち止まる
    ifelse [pcolor] of patch-ahead 1 = black [
      ;; 壁の手前なら何もしない
    ] [
      ;; 3. 周辺に人がいるかチェック
      let obstacles other turtles in-radius person-range

      ifelse any? obstacles [
        ;; 目の前に人がいたら少しだけ避ける
        let target-obstacle one-of obstacles
        face (target-obstacle)
        rt 180                 
        fd 0.15           
        rt (random 60 - 30)    
      ] [
        ;; 周りにお互いが全くいなければ目的地に向かって進む
        forward 0.15
      ]
    ]
  ]
  
  ;;==========================
  ;; 電車到着
  ;;==========================
  if ticks > 0 and ticks mod 300 = 0 [

    let arriving-passengers train-passenger-min
    if train-passenger-random > 0 [
      set arriving-passengers arriving-passengers + random train-passenger-random
    ]
    create-passengers arriving-passengers

  ]

  tick
end

to create-passengers [n]

  create-turtles n [

    set shape "person"
    set size 1.5
    

    ;; ホーム上にランダム配置
    setxy (random-float 24 - 12) (-13 - random-float 2)
    
    ;; 設定した割合でエスカレータ利用者と階段利用者に分ける
    set uses-escalator? (random-float 1 < escalator-ratio)

    ifelse uses-escalator? [
      set color blue
      set route-xcor 5
      set route-pcolor turquoise
    ] [
      set color red
      set route-xcor 0
      set route-pcolor brown
    ]

    ;; 選択した経路の入口を目指す
    set destination patch route-xcor route-entry-ycor
    set entered-route? false
    
    set enter-time ticks

  ]
end

to-report average-time
  if exit-count = 0 [
    report 0
  ]
  report total-time / exit-count
end
