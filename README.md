https://elm-lang.org/
まずは、このサイトにアクセスして、HelloWorldと表示されるプログラムやボタンを押すことで値が増減するなどのプログラムを確認した。

次に、四角い画像を表示することを目標とする。
その際に、さっきのサイトの

![](https://i.imgur.com/nVJH1kr.png)

という画面の「More!」を選択すると

![](https://i.imgur.com/LmkV2o5.png)

この画面が表示される。この中の左上にあるHTMLの中のShapesを選択すると図形の表示に関するプログラムが表示される。その中から四角形を表示する部分だけを切り取る。

```elm=

import Html exposing (Html)
import Svg exposing (..)
import Svg.Attributes exposing (..)


main : Html msg
main =
  svg
    [ viewBox "0 0 400 400"
    , width "400"
    , height "400"
    ]
    [ rect
        [ x "100"
        , y "10"
        , width "40"
        , height "40"
        , fill "green"
        , stroke "black"
        , strokeWidth "2"
        ]
        []
    ]
    
```

![](https://i.imgur.com/Fyv4faN.png)

するとこのような四角形が表示される。

次に、プログラムの内容を少し変えてみる。

![](https://i.imgur.com/5NNMbPx.png)

x,yの値を変更したら座標が変化した。

![](https://i.imgur.com/Yo0lGE5.png)


width,heightの値を変更したら大きさが変化した。

![](https://i.imgur.com/KMys446.png)

fillの中身を変更したら色が変化した。

![](https://i.imgur.com/k8AW4IQ.png)

strokeの中身を変更したら縁の色が変化した。

![](https://i.imgur.com/kqwFS2F.png)

strokewidthの値を変更したら縁の太さが変化した。

<br>

これで四角形の表示は可能になった。

<br>

次に、マウスの座標を取得する。
elmで書かれたマウスの座標を取得するプログラムを見つけたため、今回はそれを参考とする。

```elm=
import Browser
import Html exposing (..)
import Html.Events exposing (on)
import Html.Attributes exposing (style)
import Json.Decode exposing (map2, field, int)

main =
  Browser.sandbox
    { init = init
    , update = update
    , view = view
    }

type alias Model = { x: Int , y: Int }

init : Model
init = { x = 0 , y = 0 }

type Msg = Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Move x y -> {x = x, y = y}

view : Model -> Html Msg
view model =
  div []
    [ span
        []
        [ text ("(" ++ (String.fromInt model.x) ++ ", " ++ String.fromInt model.y ++ ")") ]
    , div
        [ style "background-color" "gray"
        , style "height" "80vh"
        , on "mousemove" (map2 Move (field "clientX" int) (field "clientY" int))
        ]
        []
    ]
    
```

このプログラムを実行すると、灰色のところにマウスカーソルがある際に、その座標を表示してくれる。

![](https://i.imgur.com/fNNJVhP.png)

次に、マウスをクリックすることで何かしらのアクションが起きるものを作る。
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
しかし、図形をクリックした際に、どこをつかんでも図形の左上をつかんでしまう。なので、それを修正する。

修正したのが以下のコードである。

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

これで、図形のドラッグ＆ドロップが完成した。
