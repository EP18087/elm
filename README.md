# 第10回　EP18087　林　佳吾

前回までで、マウス座標の取得はできるようになった。
今回、ドラッグ＆ドロップの実装まで頑張りたい。

まず、マウスが押されたときに色が変化し、離すと色が戻るものをつくる。

<details>
<summary>ソースコード</summary>

```elm=

import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = String

init : Model
init = "gray"

type Msg = NoOp
         | Down
         | Up

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> "red"
    Up -> "gray"
    _ -> model

view : Model -> Html Msg
view model =
  div[]
  [ div
    [ style "position" "relative"
    , style "width" "1000px"
    , style "height" "1000vh"
    , style "background" model
    , onMouseDown Down
    , onMouseUp Up
    ]
    []
  ] 
  
```
</details>

<br>

マウスを押していない間は灰色の画面が、マウスを押している間は赤い画面が表示できるようになった。

![](https://i.imgur.com/fuKGcfH.png)

![](https://i.imgur.com/pCDqyE7.png)

<br>

（しかし、マウスを押している間に赤く変化するのはうまくいったが、マウスを離す際に、色のついているところで離さないと色がずっと赤くなりっぱなしなので要修正？）

この考え方と、先週までの分を組み合わせてドラッグ＆ドロップを実装する。まず、上のものに四角形を追加する。

<details>
<summary>ソースコード</summary>

```elm=

import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = String

init : Model
init = "gray"

type Msg = NoOp
         | Down
         | Up

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> "red"
    Up -> "gray"
    _ -> model

view : Model -> Html Msg
view model =
  div[]
  [ div
    [ style "position" "relative"
    , style "width" "1000px"
    , style "height" "1000vh"
    , style "background" model
    , onMouseDown Down
    , onMouseUp Up
    ]
    []
  , div
    [ style "background-color" "black" 
    , style "height" "50px"
    , style "width" "50px"
    , style "position" "absolute"
    , style "left" ("50px")
    , style "top" ("50px")
    ]
    []
  ]
  
```

</details>

![](https://i.imgur.com/MbLgWP0.png)

追加した。（四角形をクリックしても背景の色は変わらず、それ以外のところをクリックすると色が変化する。）

次にこの四角形がマウスを追尾するようにする。

<details>
<summary>ソースコード</summary>

```elm=

import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = {color:String, x:Int, y:Int}

init : Model
init = {color = "white", x = 50, y = 50}

type Msg = Down
         | Up
         | Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Move x y -> {model | x = x, y = y}
    Down -> {model | color = "red"}
    Up -> {model | color = "gray"}

view : Model -> Html Msg
view model =
  div[]
  [ span
    []
    [ text ("(" ++ (String.fromInt model.x) ++ ", " ++ String.fromInt model.y ++ ")") ]
  , div
    [ style "position" "absolute"
    , style "width" "600px"
    , style "height" "80vh"
    , style "background" model.color
    , onMouseDown Down
    , onMouseUp Up
    , onMouseMove Move
    ]
    []
  , div
    [ style "background-color" "black" 
    , style "height" "50px"
    , style "width" "50px"
    , style "position" "absolute"
    , style "left" (String.fromInt model.x ++ "px")
    , style "top" (String.fromInt model.y ++ "px")
    , onMouseMove Move
    ]
    []
  ]

onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f =
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))
  
```
  
  </details>
  
これで、勝手に四角形がマウスカーソルを追尾するようになった。なので、後はマウスが押されている間にカーソルを追いかけて、離したときにそれを止めるようにすればドラッグ＆ドロップが出来る。

<details>
<summary>ソースコード</summary>

```elm=

import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = {x:Int, y:Int, dragflag:Bool}

init : Model
init = {x = 50, y = 50, dragflag = False}

type Msg = Down
         | Up
         | Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | dragflag = True}
    Up -> {model | dragflag = False}
    Move x y ->
      {model
        | x = if (model.dragflag == True) then x
              else model.x
        , y = if (model.dragflag == True) then y
              else model.y
       }
              
view : Model -> Html Msg
view model =
  div[]
  [ span
    []
    [ text ("(" ++ (String.fromInt model.x) ++ ", " ++ String.fromInt model.y ++ ")") ]
  , div
    [ style "position" "absolute"
    , style "width" "1000px"
    , style "height" "80vh"
    , style "background" "gray"
    , onMouseMove Move
    , onMouseUp Up
    ]
    []
  , div
    [ style "background-color" "black" 
    , style "height" "50px"
    , style "width" "50px"
    , style "position" "absolute"
    , style "left" (String.fromInt model.x ++ "px")
    , style "top" (String.fromInt model.y ++ "px")
    , onMouseDown Down
    , onMouseUp Up    
    , onMouseMove Move
    ]
    []
  ]

onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f =
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))

```

</details>

上のコードでドラッグ＆ドロップはできるようになった。
しかし、問題点がいくつかある。

１. マウスを早く動かすとドラッグ中じゃないのにマウスカーソルを追従してしまうことがある。
2.　offsetを使用していない。
3.　コードが短くまとまっていない。

なので、次回までにこの3つの問題を解決しようと思う。


解決

<details>
<summary>ソースコード</summary>

```elm=
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = {x:Int, y:Int, dragflag:Bool, ox:Int, oy:Int}

init : Model
init = {x = 50, y = 50, dragflag = False, ox = 0, oy = 0}

type Msg = Down
         | Up
         | Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | dragflag = True}
    Up -> {model | dragflag = False}
    Move x y ->
      {model
        | x = if (model.dragflag == True) then x - model.ox
              else model.x
        , y = if (model.dragflag == True) then y - model.oy
              else model.y
        , ox = if (model.dragflag == True) then model.ox
              else x - model.x
        , oy = if (model.dragflag == True) then model.oy
              else y - model.y              
       }
              
view : Model -> Html Msg
view model =
  div[]
  [ span
    []
    [ text ("(" ++ (String.fromInt model.x) ++ ", " ++ String.fromInt model.y ++ ")") ]
  , div
    [ style "position" "absolute"
    , style "width" "1000px"
    , style "height" "80vh"
    , style "background" "gray"
    , onMouseMove Move
    , onMouseUp Up
    ]
    []
  , div
    [ style "background-color" "black" 
    , style "height" "50px"
    , style "width" "50px"
    , style "position" "absolute"
    , style "left" (String.fromInt model.x ++ "px")
    , style "top" (String.fromInt model.y ++ "px")
    , onMouseDown Down
    , onMouseUp Up    
    , onMouseMove Move
    ]
    []
  ]

onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f =
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))
 
```
</details>
