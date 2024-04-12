ConstraintsHolder is a tiny Auto Layout framework based on modern Anchors API that simplifies working with constraints by abstracting away their storage. It is especially useful in case you change constraints often such as in applications with dynamic UI.

### Installation
To install via Swift Package Manager (SPM) simply do the following:
1. From Xcode, select from the menu File > Add Package Dependency
2. Paste the URL https://github.com/leofriskey/ConstraintsHolder

### Why use ConstraintsHolder
Consider a following common example:
``` Swift
private let vw = UIView()
private var vwTopConstraint: NSLayoutConstraint?

...

override func viewDidLoad() {
    super.viewDidLoad()

    vw.translatesAutoResizingMaskIntoConstraints = false
    view.addSubview(vw)

    let topConstraint = vw.topAnchor.constraint(equalTo: view.topAnchor, constant: 20)
    vwTopConstraint = topConstraint

    NSLayoutConstraint.ativate([
        topConstraint,
        // other constraints
    ])
}

...

func changeConstraint() {
    vwTopConstraint?.constant = 50
    view.layoutIfNeeded()
}

```

As shown above if we wish to change the constraint affecting our view, we need to store reference to it in our ViewController.
And since in most modern apps we often have more than one moving parts this whole thing would very fast become cumbersome and bloat `UIViewController` with all references to different constraints affecting different views:
``` Swift
private var vw1TopConstraint: NSLayoutConstraint?
private var vw2LeadingConstraint: NSLayoutConstraint?
...
private var vw5LeadingConstraint: NSLayoutConstraint?
```

Another example would be de-activating, changing and then re-activating constraint which too requires that we store a reference to that constraint somewhere in our `UIView`/`UIViewController` and then we would have to unwrap it before activation.

This boilerplate and error-prone pattern can be avoided using **ConstraintsHolder** framework

### Usage
``` Swift
private let vw = UIView()

...

override func viewDidLoad() {
    super.viewDidLoad()

    vw.translatesAutoResizingMaskIntoConstraints = false
    view.addSubview(vw)

    vw.updateConstraints { hodler in
        hodler.top = vw.topAnchor.constraint(equalTo: view.topAnchor, constant: 20)

        hodler.activate([
            \.top
        ])
    }
}

...

func changeConstraint() {
    vw.updateConstraints { hodler in
        hodler.top?.constant = 50
        view.layoutIfNeeded()
    }
}

```

This will do the very same thing but we no longer need to store a reference to `topAnchor` constraint inside ViewController because we assign it to view's `holder` - constraints container. 

Besides convinience and reducing boilerplate there are other benefits to this approach such as:

1) Error-prone constraints assignment:
``` Swift

    vw.updateConstraints { hodler in
        hodler.top = vw.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20) 
        // fatalError: Can't assign value of ConstraintType.leading to variable of ConstraintType.top

        hodler.activate([
            \.top
        ])
    }

```
2) Error-prone constraint activation/de-activation:
``` Swift
    vw.updateConstraints { hodler in
        hodler.top = vw.topAnchor.constraint(equalTo: view.topAnchor, constant: 20) 

        hodler.activate([
            \.bottom
        ])
        // fatalError: keyPath passed to activate() contained nil value constraint
    }
```

``` Swift
    vw.updateConstraints { hodler in
        hodler.deactivate([
            \.bottom
        ])
        // fatalError: keyPath passed to deactivate() contained nil value constraint
    }
```
3) Since framework uses keyPaths - you can't mistakenly activate/deactivate another constraint thad doesn't affect your view like in this example below:
``` Swift
private var vw1TopConstraint: NSLayoutConstraint?
private var vw2TopConstraint: NSLayoutConstraint?

...

override func viewDidLoad() {
    super.viewDidLoad()

    ...

    guard let vw2TopConstraint else { return }

    NSLayoutConstraint.activate([
        vw2TopConstraint // should've been vw1
    ])
}
```

4) Consice and beautiful code. Update view-bound constraints from anywhere!
``` Swift
// just an example piece of code from one of my apps
...

assetsTotalValueLabel.updateConstraints { holder in
    // deactivate old constraints
    holder.deactivate([
        \.centerY,
        \.leading
    ])
    
    // replace old constraints with new
    holder.centerY = assetsTotalValueLabel.centerYAnchor.constraint(equalTo: navBar.centerYAnchor)
    holder.leading = assetsTotalValueLabel.leadingAnchor.constraint(equalTo: smallTitleView.trailingAnchor, constant: 10)
    
    // activate new constraints
    holder.activate([
        \.centerY,
        \.leading
    ])
}
```