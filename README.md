# widget_kit_tutorial

# Terms

- Timeline
- Entry
- Snapshot
- Updates 

# Timeline

It provides the timeline. 
A type that advises WidgetKit when to update a widget's display. 

- placeholder 

When system doesn't have data, it shows with Provider.(dummy data)

- getSnapshot 

The lastest version of widget. 
Don't do any network calls on `getSnapshot`.

- getTimeline

This is where timeline actually gets created. 
`Entry` is data. 
Timeline is an array of entries. 
You can give task like `update every hour` here.

# EntryView: View 

SwiftUI View.
Create your own view.

# Widget: Widget

Acutal widget

# Configuration 

- Static View 

User can configure static view. 
Don't misunderstand terms. 'CountDown Timer' is also static view. 

- IntentConfiguration 

User can hold onto widget and customize it. 

# Code

### Bundle with multiple widgets 
```swift
//
//  MonthlyWidgetBundle.swift
//  MonthlyWidget
//
//  Created by paige shin on 2022/12/25.
//

import WidgetKit
import SwiftUI

@main
struct MonthlyWidgetBundle: WidgetBundle {
    var body: some Widget {
        NewWidget()
        NewAnotherMonthlyWidget()
        // Add Multiple Widgets
        MonthlyWidget()
    }
}

```
### Widgets with same provider
```swift
//
//  MonthlyWidget.swift
//  MonthlyWidget
//
//  Created by paige shin on 2022/12/25.
//

import WidgetKit
import SwiftUI

struct NewProvider: TimelineProvider {
    func placeholder(in context: Context) -> NewEntry {
        NewEntry(date: Date())
    }

    func getSnapshot(in context: Context, completion: @escaping (NewEntry) -> ()) {
        let entry = NewEntry(date: Date())
        completion(entry)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [NewEntry] = []


        // Generate a timeline consisting of seven entries a day apart, starting from the currente date.
        let currentDate = Date()
        for dayOffset in 0..<7 {
            let entryDate = Calendar.current.date(byAdding: .day, value: dayOffset, to: currentDate)!
            let startOfDate = Calendar.current.startOfDay(for: entryDate)
            let entry = NewEntry(date: startOfDate)
            entries.append(entry)
        }
        
        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        /*
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate)
            entries.append(entry)
        }
         */
        
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct NewEntry: TimelineEntry {
    let date: Date
}

struct NewMonthlyWidgetEntryView : View {
    
    @Environment(\.widgetFamily) var family
    
    var entry: NewProvider.Entry

    var body: some View {
        ZStack {
            ContainerRelativeShape()
                .fill(.gray.gradient)
            
            VStack {
                HStack(spacing: 4) {
                    Text("☃️")
                        .font(.title)
                    Text("WIDGET WITH SAME PROVIDER")
                        .font(.title3)
                        .fontWeight(.bold)
                        .minimumScaleFactor(0.4)
                        .foregroundColor(.black.opacity(0.6))
                    Spacer()
                }
                
                Text(entry.date.dayDisplayFormat)
                    .font(.system(size: 80, weight: .heavy))
                    .foregroundColor(.white.opacity(0.8))
            }
            .padding()
            
        }
    }
}

struct NewWidget: Widget {
    let kind: String = "NewWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: NewProvider()) { entry in
            NewMonthlyWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Monthly Style Widget")
        .description("The theme of the widget cahnges based on month.")
//        .supportedFamilies([.systemSmall]) // only support small
    }
}

struct NewAnotherMonthlyWidget: Widget {
    let kind: String = "NewAnotherMonthlyWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: NewProvider()) { entry in
            ZStack {
                ContainerRelativeShape()
                    .fill(.blue.gradient)
                
                VStack {
                    Text("WIDGET2 WITH SAME PROVIDER")
                        .font(.title3)
                        .fontWeight(.bold)
                        .minimumScaleFactor(0.4)
                        .foregroundColor(.black.opacity(0.6))
                }
                .padding()
                
            }
        }
        .configurationDisplayName("New Monthly Style Widget")
        .description("New the theme of the widget cahnges based on month.")
//        .supportedFamilies([.systemSmall]) // only support small
    }
}


extension Date {
    var weekDisplayFormat: String {
        self.formatted(.dateTime.weekday(.wide))
    }
    var dayDisplayFormat: String {
        self.formatted(.dateTime.day())
    }
}

```

### Widgets with different provider

```swift
//
//  MyWidget.swift
//  WidgetTutorial
//
//  Created by paige shin on 2022/12/25.
//

import WidgetKit
import SwiftUI

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date())
    }

    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []


        // Generate a timeline consisting of seven entries a day apart, starting from the currente date.
        let currentDate = Date()
        for dayOffset in 0..<7 {
            let entryDate = Calendar.current.date(byAdding: .day, value: dayOffset, to: currentDate)!
            let startOfDate = Calendar.current.startOfDay(for: entryDate)
            let entry = SimpleEntry(date: startOfDate)
            entries.append(entry)
        }

        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        /*
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate)
            entries.append(entry)
        }
         */

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
}

struct MonthlyWidgetEntryView : View {

    @Environment(\.widgetFamily) var family

    var entry: Provider.Entry

    var body: some View {
        ZStack {
            ContainerRelativeShape()
                .fill(.white.gradient)

            VStack {
                HStack(spacing: 4) {
                    Text("☃️")
                        .font(.title)
                    Text("WIDGET3 WITH DIFFERENT PROVIDER")
                        .font(.title3)
                        .fontWeight(.bold)
                        .minimumScaleFactor(0.4)
                        .foregroundColor(.black.opacity(0.6))
                    Spacer()
                }

                Text(entry.date.dayDisplayFormat)
                    .font(.system(size: 80, weight: .heavy))
                    .foregroundColor(.white.opacity(0.8))
            }
            .padding()

        }
    }
}

struct MonthlyWidget: Widget {
    let kind: String = "MonthlyWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            MonthlyWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Monthly Style Widget")
        .description("The theme of the widget cahnges based on month.")
    }
}

struct MonthlyWidget_Previews: PreviewProvider {
    static var previews: some View {
        MonthlyWidgetEntryView(entry: SimpleEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}


```
