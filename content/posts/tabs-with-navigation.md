---
title: "Tabs with NavigationStacks: Deeper Programmatic Navigation"
date: 2024-09-22T12:00:00-07:00
draft: false
---
 
There are some great developer posts about programmatic navigation using `NavigationStack` with `NavigationPath` such as [this one](https://buresdv.substack.com/p/programmatic-navigation-in-swiftui?r=2vioyx&triedRedirect=true) I recently read by David Bureš. This is great for apps that need navigation depth but not breadth. By breadth, I mean apps whose scope contains categories of screens that require a `TabView`. If your app can benefit from the breadth of using tabs in addition to navigation stacks that give you the depth, you can still achieve programmatic navigation as well.

If you want to browse or try this example code yourself, the [whole project](https://github.com/drmarkpowell/PocketMonsterIndex) is available (requires iOS 18 or macOS 15 to run).

### Catching all the tabs and nav stacks

For this post, I created an example app with two tabbed views. The first tab displays a Grid view of Pokémon (or a Pokédex if you like). The Grid supports a `NavigationStack` that displays a detail view if you tap on a grid cell view. The second tab displays a Grid view of Berries (or an inventory if you prefer). Tapping on a grid cell view here pushes a berry detail view onto the second tabbed view's `NavigationStack`.

Let's start by looking at the main screen that creates a `TabView`:

```swift
@Environment(AppModel.self) var appModel

var body: some View {
    @Bindable var appModel = appModel
    TabView(selection: $appModel.selectedTabName) {
        Tab(
            TabName.pokedex.title,
            systemImage: "square.grid.3x3",
            value: TabName.pokedex
        ) {
            PokedexScreen()
        }

        Tab(
            TabName.berries.title,
            systemImage: "carrot.fill",
            value: TabName.berries
        ) {
            BerriesScreen()
        }
    }
    .tabViewStyle(.sidebarAdaptable)
}
```

This `TabView` uses the newest SwiftUI API for iOS 18. If you're using the iOS 17 API or earlier, you need to use `.tag` instead of the `value` to [support tab selection](https://www.hackingwithswift.com/quick-start/swiftui/how-to-embed-views-in-a-tab-bar-using-tabview) (Thanks, Paul!). I also threw in the iPadOS 18 `.sidebarAdaptable` tab view style, which Stewart Lynch made a [great tutorial](https://youtu.be/hIoxphFMjYw?si=AQfciJa4qmleCVoq) about, as well as tab customization and reordering (omitted here for brevity).

The `AppModel` is our application state model class (ViewModel in the MVVM sense if you like) that will contain our selected tab and navigation paths for each tab: The `TabView` above binds the `selectedTab` to this var in `AppModel`. Let's look at that next.

### The AppModel Class

```swift
@Observable
class AppModel {
    var selectedTabName = TabName.pokedex
    var pokedexTab = TabModel(name: .pokedex)
    var berriesTab = TabModel(name: .berries)
    
    func navigateTo(category: String, index: Int) {
        if category == berriesTab.name.rawValue {
            berriesTab.path = NavigationPath([index])
            selectedTabName = .berries
        } else if category == pokedexTab.name.rawValue {
            pokedexTab.path = NavigationPath([index])
            selectedTabName = .pokedex
        }
    }
}

@Observable
class TabModel {
    let name: TabName
    var path = NavigationPath()
    init(name: TabName) {
        self.name = name
    }
}

enum TabName: String, CaseIterable {
    case pokedex
    case berries
    var title: String { rawValue.capitalized }
}
```

Using an enumeration to define the tabs the app supports is generally a good idea and useful for programmatic navigation. `TabModel` is a custom `@Observable` class for capturing each tab instance's unique name along with its `NavigationPath` so that we can drive that programmatically as well.

Here's a simplified look at the Pokédex view (the first of the tabbed views):

```swift
struct PokedexScreen: View {
    @Environment(AppModel.self) var appModel
 
    var body: some View {
        @Bindable var appModel = appModel
        
        NavigationStack(path: $appModel.pokedexTab.path) {
            LazyVGrid(columns: [...]) {
                ForEach(pokemon) { pokemon in
                    Button {
                        appModel.pokedexTab.path.append(pokemon.id)
                    } label: {
                        // ... cut for legibility
                    }
                }
            }
            .navigationTitle(Text(TabName.pokedex.title))
            .navigationDestination(for: Int.self) { id in
                PokemonScreen(id: id)
            }
        }
    }
```

In this view, the `NavigationPath` is bound to the path for its corresponding `TabModel` in the `AppModel`. When the user taps a grid cell view, the ID of the Pokémon is pushed onto the path and the app navigates to a newly created `PokemonScreen` (view). The `BerriesScreen` works similarly, navigating to a new `BerryScreen` when a Berry grid cell view is tapped.

### Flexible Navigation

To show the flexibility of how this programmatic navigation can be used to jump into any content in the app, let's implement deep linking:

```swift
@main
struct PocketMonsterIndexApp: App {
    @State private var appModel = AppModel()

    var body: some Scene {
        WindowGroup {
            TabsView()
                .environment(appModel)
                .onOpenURL { url in
                    print("\(url.pathComponents)")
                    guard let index = Int(url.pathComponents.last ?? "") else {
                        return 
                    }
                    guard let category = url.host() else { 
                        return 
                    }
                    appModel.navigateTo(category: String(category), index: index)
                }
        }
    }
}
```

The `.onOpenURL(URL)` implemented here will let the app respond to opening links from a browser or other apps. For example, I chose to respond to links like [pmi://pokedex/99](pmi://pokedex/99) and [pmi://berries/16](pmi://berries/16). When the app responds, it looks at the url host and path values. The host value is expected to be one of the supported tab names (pokedex, berries) and the path is expected to contain an index integer value. The `AppModel.navigateTo(category, index)` function will set the selected tab and its corresponding NavigationPath index, cleanly navigating you to the details.

I think this approach would also work very well with App Intents if you are coming in from a Shortcut, Spotlight or Siri. This wasn't space for all that in this post, however. If you would like to see how that would look, [send me a message](https://iosdev.space/@drmarkpowell) on Mastodon and let me know.

If you want to see the [whole Xcode project](https://github.com/drmarkpowell/PocketMonsterIndex), browse the code or give it a try (iOS 18 or macOS 15 required to run).
