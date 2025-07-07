# タイトル
Swift Charts の軸に良い感じで日時を表示する

# 内容
`Swift Charts`を使うことで簡単にグラフを作成することが出来ます。
一方で、細かい調整を行うには対象のAPIを使う必要がありますが、ドキュメントだけでは、どのように作用するかが不明なことが多いです。
そこで今回はグラフに不可欠なラベルの調整方法の実例を紹介し、理解していここうと思います。

## データ構造
今回の記事では以下のデータ構造を使って説明します。
時刻とそのときの回数を保持するだけの簡単なデータ構造です。

```swift
struct Record: Identifiable {
    var id: UUID = UUID()
    let date: Date
    let count: Int
}

var records: [Record] = [
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 1, hour: 18).date!, count: 10),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 2, hour: 18).date!, count: 20),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 3, hour: 18).date!, count: 5),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 4, hour: 18).date!, count: 5),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 5, hour: 18).date!, count: 5),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 6, hour: 18).date!, count: 5),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 7, hour: 18).date!, count: 5),
    Record(date: DateComponents(calendar: .current, year: 2022, month: 11, day: 8, hour: 18).date!, count: 5),
]
```

## デフォルトのグラフ表示
まずは上記のデータを `Charts` を使ってグラフ表示してみます。

```swift
struct ContentView: View {
    var body: some View {
        Chart(records) { record in
            PointMark(
                x: .value("Date", record.date),
                y: .value("Count", record.count)
            )
        }
        .frame(height: 200)
        .padding(.horizontal)
    }
}
```

表示されるグラフは以下のようになります。
（images配下のimage1）


次にこのコードを修正して、どのようにX軸が変化するかを見ていきます。

## X軸に関する表示内容を調整
X軸ラベルを調整するには、 `.chartXAxis` を使用します。
以下のコードでは、上記コードに`.chartXAxis` を追加しています。
 
```swift
struct ContentView: View {
    var body: some View {
        Chart(records) { record in
            PointMark(
                x: .value("Date", record.date),
                y: .value("Count", record.count)
            )
        }
	// ここでX軸ラベルを調整する
        .chartXAxis {
            AxisMarks(values: .automatic)
        }
        .frame(height: 200)
        .padding(.horizontal)
    }
}
```

このコードでグラフを表示すると、コードの変更前と変わらないグラフが表示されます。
つまり、指定がない場合は

```swift
.chartXAxis {
    AxisMarks(values: .automatic)
}
```

を明示的に指定した状態と同等になります。

さらに変更を加えることで、表示内容を変更していくことができます。
`.chartXAxis` を以下のように修正します。

```swift
.chartXAxis {
    AxisMarks(values: .automatic) { date in
        AxisGridLine()
    }
}
```

表示結果は以下の通りです。
（images配下のimage2）


`AxisGridLine()` を明示的に指定したことで、他の要素が消えてしまいました。
この状態ではグラフとして成立していないので、更に以下のように修正します。

```swift
.chartXAxis {
    AxisMarks(values: .automatic) { date in
        AxisGridLine()
	AxisTick()
	AxisValueLabel(format: .dateTime.day())
    }
}
```

表示結果は以下の通りです。
（images配下のimage3）

`.chartXAxis` を指定しない状態とほぼ同じ内容が表示出来ました。
違いは、X軸のラベルが日付だけになっている点です。
これは `AxisValueLabel(format: .dateTime.day())` と指定したことで、日付だけを表示するようになった結果です。

この部分を調整することで、軸ラベルを変更することが出来ます。
例えば、日付の次に月と年を表示したい場合は、以下のようなコードになります。

```swift
.chartXAxis {
    AxisMarks(values: .automatic) { date in
        AxisGridLine()
	AxisTick()
	AxisValueLabel(format: .dateTime.day().month().year())
    }
}
```



このように `.chartXAxis` 内で、 `AxisMarks` を調整することで、X軸に関する表示内容を調整することができます。

## X軸ラベルの表示間隔の調整方法
ここまでは、X軸に関する表示内容の調整方法を紹介してきました。
次に、X軸のラベルの表示間隔を調整する方法を紹介します。

例えば今までは2日おきにラベルが配置されいましたが、これを1日おきに表示したい、という場合を考えてみます。
`AxisMarks(values: .automatic)` では、ラベルの表示間隔が自動で調整されてしまうため、以下のように修正をします。

```swift
.chartXAxis {
    AxisMarks(values: .stride(by: .day, count: 1))
}
```

このコードの変更によって、以下のようにX軸のラベルの間隔を1日おきにすることが出来ました。
（images配下のimage4）


もしX軸ラベルの間隔を3日おきにしたければ、 `count: 3` とすることで可能です。

## まとめ
比較的データの日時の範囲が小さい場合は特に指定しなくても、 `Charts` 側で良い感じに軸の表示内容を調整してくれます。
一方で、日時が広くなったり、アプリの性質上細かくラベルを表示したい場合などは、コードで適切に設定する必要があります。
その際には `AxisMarks(values: .stride(by: , count: ))` を使って、表示内容を整理するのが良さそうです。