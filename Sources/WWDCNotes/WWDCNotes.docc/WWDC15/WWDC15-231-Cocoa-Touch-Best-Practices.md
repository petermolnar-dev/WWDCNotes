# Cocoa Touch Best Practices

Small improvements can make a big impact on your iOS app's performance and launch time. Learn best practices to optimize your UIKit-based applications. See how to future-proof your app for what may be next, and walk through numerous real world examples for more responsive and dynamic collection and table views.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/231", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(antonio081014)
   }
}



## App Lifecycle

### Beyond App Launch, Being responsive to every input

- Put on the work to the background queue, when you are done with it, put it back to main queue.

### Leverage Frameworks

- Target two most recent major releases

## Views and View Controllers

### Layout to Proportions

- Avoid hard-coded layout values

### Size Classes

- Some size thresholds trigger major change
- Properties, not Tags. Keep a property instead.

### Leverage `UIViewControllerTransitionCoordinator`

- Animate alongside a transition
- Get accurate completion timing
- Support interactive and cancelable animations

## Auto Layout

### Modify Constraints Efficiently

- Identify constraints that get changed, added, or removed 
- Unchanged constraints are optimized
- **Avoid removing all constraints**
- Use explicit constraint references

### Constraint Specificity 

- De-duplicating constraints
  - Duplicates are implied by existing constraints 
  - Duplicates cause excess work in layout engine

- Create flexible constraints
  - Avoid hard-coded values
  - Describe constraints using bounds

- Fully specify constraints
- Testing and debugging
  - `-[UIView hasAmbiguousLayout]`, when called on `UIWindow`, returns result for entire view tree

-  
  - `-[UIView _autolayoutTrace]`, Call this method on a view. It returns a string with diagnostic information about the entire view hierarchy containing that view. Ambiguous views are labeled, and so are views that have translatesAutoresizingMaskIntoConstraints set to YES.
  - This could be used in Unit Test, try call `-[UIView hasAmbiguousLayout]` first, if it returns `YES`, then call `-[UIView _autolayoutTrace]` to help figure out the cause.
  
## Table and Collection Views

### Self-Sizing Cells

- Fully specify constraints
- Width = input; height = output.
- Tip: add a constraint to cell's content view, height

### Animating Height Changes

Naive way: update the model, then call `tableView.reloadData()`

Promoted way: 

- **`tableView.beginUpdates`**
- Update model
- Update cell contents, might just **call `tableView.cellForRowAtIndexPath` and update the content with corresponding model.**
- **`tableView.endUpdates`**
### Fast CollectionView Layout Invalidation
- Invalidate on bounds change
- Build [targeted invalidation context](https://developer.apple.com/documentation/uikit/uicollectionviewlayoutinvalidationcontext)
- Repeat as necessary