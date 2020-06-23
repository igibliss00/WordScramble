# Word Scramble

<img src="https://github.com/igibliss00/EasyBrowser/blob/master/README_assets/1.png" width="400">
<img src="https://github.com/igibliss00/EasyBrowser/blob/master/README_assets/2.png" width="400">
<img src="https://github.com/igibliss00/EasyBrowser/blob/master/README_assets/3.png" width="400">

## Installing

To clone the project

```
git clone https://github.com/igibliss00/WordScramble.git
```

## Features

### Bundle

Bundle in Swift is a way Xcode constructs a directory with executable code and the resources that the code uses.  This bundle includes apps, frameworks, plug-ins, and other contents.  Depending the project, the exact structure is going to be different, but the bundle object searches for the item with the parameter provided by you, which means you don't need to know what the exact structure looks like 

In this project, we fetch the URL of the resource file of my choice in order to use the content in an array.

```
if let startWordsURL = Bundle.main.url(forResource: "start", withExtension: "txt") {
    if let startWords = try? String(contentsOf: startWordsURL) {
        allWords = startWords.components(separatedBy: "\n")
    }
    
    if allWords.isEmpty {
        allWords = ["silkworm"]
    }
}
```

Here we are using "Bundle.main.url" to fetch the file "start.txt".  This file contains over 12,000 8-letter words which will be used in the game.  The main bundle "represents the bundle directory that contains the currently executing code," as explained in the Apple's [documentation](https://developer.apple.com/documentation/foundation/bundle).  It basically represents the current bundle we're using and the resources that are shipped with the app.  

We use the "String()" method to convert the content of the file into a string format and then a string method called "components(separatedBy:)" method to split up the words where there is a new line and add them to an array.

### Table View Reload

We use the following code to reload the table:

```
tableView.reloadData()
```

This command calls the "numberOfRowsInSection" function as well as the "cellForRowAt" function again.

### Table View Insert

Contrary to above, the following code uses the table view insert in order to display a newly added item on the table. 

```
usedWords.insert(answer, at: 0)
let indexPath = IndexPath(row: 0, section: 0)
tableView.insertRows(at: [indexPath], with: .automatic)
```

The reason being, first, that reloading the entire table for a single insert is expensive and, second, we want to utilize the insertion animation from the table view.  The "automatic" provides the standard animation within a table when a new row is inserted from the top.   

### Text Checker

The objective of this game is to use the characters from a given word and re-arrange them to create another legitimate word.  In order to qualify whether the newly created word is a real word or not, we use the the "UITextChecker" type:

```
func isReal(word: String) -> Bool {
    let checker = UITextChecker()
    let range = NSRange(location: 0, length: word.utf16.count)
    let misspelledRange = checker.rangeOfMisspelledWord(in: word, range: range, startingAt: 0, wrap: false, language: "en")

    return misspelledRange.location == NSNotFound
}
```

"NSRange" is to obtain the full gamut of the given word from 0 to "word.utf16.count".  Instead of using "word.count", we use UTF-16 (16-bit Unicode Transformation Format) because "NSRange" predates Swift and was written in Objective-C.  We have to provide the string format that Objective-C understands as a parameter.  Finally, the "rangeofMisspelledWord" method returns the location of the misspelling which we are going to be comparing with "NsNotFound".

### Exiting a Function

When there are nested if-else statements, it's important to know when to exit the function depending on when certain conditions are met.  

```
func submit(_ answer: String) {
    let lowerAnswer = answer.lowercased()
    
    let errorTitle: String
    let errorMessage:String
    
    if isPossible(word: lowerAnswer) {
        if isOriginal(word: lowerAnswer) {
            if isReal(word: lowerAnswer) {
                usedWords.insert(answer, at: 0)
                
                let indexPath = IndexPath(row: 0, section: 0)
                tableView.insertRows(at: [indexPath], with: .automatic)
                return // exit!
            } else {
                errorTitle = "Word not recognized"
                errorMessage = "You can't just make up the word, you know"
            }
        } else {
            errorTitle = "Word already used"
            errorMessage = "Be more original!"
        }
    } else {
        guard let title = title else { return }
        errorTitle = "Word not possible"
        errorMessage = "You can't spell that word from \(title.lowercased())"
    }
    
    let ac = UIAlertController(title: errorTitle, message: errorMessage, preferredStyle: .alert)
    ac.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
    present(ac, animated: true)
}
```
Here shows the nested if-else statements with multiple different ways of exiting the function.  When any of the three "if" conditions are not met "isPossible", 'isOriginal", or "isReal", the "else" statement will assign strings to the two constants, "errorTitle" and "errorMessage",  execute the "UIAlertController" and exit the function.  If, however, all three conditions are met, a new row will be inserted to the table view and exit the function.  

If the "return" statement" is not set in place, the function will not exit right away and execute the "UIAlertController".  This is logically problematic because there is no reason for a warning message to show and technically problematic because the two constants, "errorTitle" and "errorMessage" are nil.  

