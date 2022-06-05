# [Displaying Searchable Content by Using a Search Controller](https://developer.apple.com/documentation/uikit/view_controllers/displaying_searchable_content_by_using_a_search_controller#//apple_ref/doc/uid/TP40014683)

Create a user interface with searchable content in a table view.

## Overview

This sample demonstrates how to create a table view controller and search controller to manage the display of searchable content. It creates another custom table view controller to display the search results. This table view controller also acts as the presenter or provides context for the search results so they're presented within their own context.

This sample includes the optional—but recommended—[`UIStateRestoring`](https://developer.apple.com/documentation/uikit/uistaterestoring) protocol. You adopt this protocol from the view controller class to save the search bar's active state, first responder status, and search bar text and restore them when the app is relaunched.

## Create a Search Controller

Use `MainTableViewController`, a subclass of [`UITableViewController`](https://developer.apple.com/documentation/uikit/uitableviewcontroller), to create a search controller. The search controller searches and filters a set of `Product` objects and displays them in a table called `ResultsTableController`. This table controller is displayed as the user enters a search string and is dismissed when the search is complete.

``` swift
override func viewDidLoad() {
    super.viewDidLoad()

    let nib = UINib(nibName: "TableCell", bundle: nil)
    tableView.register(nib, forCellReuseIdentifier: tableViewCellIdentifier)
    
    resultsTableController =
        self.storyboard?.instantiateViewController(withIdentifier: "ResultsTableController") as? ResultsTableController
    // This view controller is interested in table view row selections.
    resultsTableController.tableView.delegate = self
    
    searchController = UISearchController(searchResultsController: resultsTableController)
    searchController.delegate = self
    searchController.searchResultsUpdater = self
    searchController.searchBar.autocapitalizationType = .none
    searchController.searchBar.delegate = self // Monitor when the search button is tapped.
    
    searchController.searchBar.scopeButtonTitles = [Product.productTypeName(forType: .all),
                                                    Product.productTypeName(forType: .birthdays),
                                                    Product.productTypeName(forType: .weddings),
                                                    Product.productTypeName(forType: .funerals)]

    // Place the search bar in the navigation bar.
    navigationItem.searchController = searchController
    
    // Make the search bar always visible.
    navigationItem.hidesSearchBarWhenScrolling = false
    
    /** Search presents a view controller by applying normal view controller presentation semantics.
        This means that the presentation moves up the view controller hierarchy until it finds the root
        view controller or one that defines a presentation context.
    */
    
    /** Specify that this view controller determines how the search controller is presented.
        The search controller should be presented modally and match the physical size of this view controller.
    */
    definesPresentationContext = true
    
    setupDataSource()
}
```

## Update the Search Results

This sample uses the [`UISearchResultsUpdating`](https://developer.apple.com/documentation/uikit/uisearchresultsupdating) protocol, along with [`NSComparisonPredicate`](https://developer.apple.com/documentation/foundation/nscomparisonpredicate), to filter search results from the group of available products. `NSComparisonPredicate` is a foundation class that specifies how data should be fetched or filtered using search criteria. The search criteria are based on what the user types in the search bar, which can be a combination of product title, year introduced, and price.

To prepare for a search, the search bar content is trimmed of all leading and trailing space characters. Then the search string is passed to the `findMatches` function, which returns the `NSComparisonPredicate` used in the search. The product list results are applied to the search results table as a filtered list.

``` swift
func updateSearchResults(for searchController: UISearchController) {
       // Update the filtered array based on the search text.
       let searchResults = products

       // Strip out all the leading and trailing spaces.
       let whitespaceCharacterSet = CharacterSet.whitespaces
       let strippedString =
           searchController.searchBar.text!.trimmingCharacters(in: whitespaceCharacterSet)
       let searchItems = strippedString.components(separatedBy: " ") as [String]

       // Build all the "AND" expressions for each value in searchString.
       let andMatchPredicates: [NSPredicate] = searchItems.map { searchString in
           findMatches(searchString: searchString)
       }

       // Match up the fields of the Product object.
       let finalCompoundPredicate =
           NSCompoundPredicate(andPredicateWithSubpredicates: andMatchPredicates)

       let filteredResults = searchResults.filter { finalCompoundPredicate.evaluate(with: $0) }

       // Apply the filtered results to the search results table.
       if let resultsController = searchController.searchResultsController as? ResultsTableController {
           resultsController.filteredProducts = filteredResults
           resultsController.tableView.reloadData()

           resultsController.resultsLabel.text = resultsController.filteredProducts.isEmpty ?
               NSLocalizedString("NoItemsFoundTitle", comment: "") :
               String(format: NSLocalizedString("Items found: %ld", comment: ""),
                      resultsController.filteredProducts.count)
       }
   }
```
