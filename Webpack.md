### Εισαγωγή
Το webpack είναι ένας static module bundler για javascript εφαρμογές, δημιουργεί δηλάδή εσωτερικά ένα depedency graph.

Για να γίνει κατανοητό αυτό, πρέπει να δοθεί ο ορισμός του **depedency**. Ένα depedency λοιπόν, δείχνει την σχέση που υπάρχει όταν ένα αρχείο εξαρτάται από ένα άλλο.

Το webpack λοιπόν ξεκινάει από μια λίστα με modules που ορίζονται από τον χρήστη και πάνω σε αυτά δημιουργεί το depedency graph, το οποίο στο τέλος κάνει bundle (ένα ή περισσότερα), όλα τα modules που χρειάζεται η εφαρμογή, έτσι ώστε να φορτωθούν από τον browser.

Τρέχει πάνω στο Node js, σε όλες τις εκδόσεις του πάνω από την 8.00 και σε όλους τους ES5 compliant browsers.

### Entry point
Είναι το αρχείο από το οποίο θα ξεκινήσει η δημιουργία του depedency graph. Το default είναι το './src/index.js' αλλά μπορεί να αλλάξει εύκολα στο configuration, μέσω του πεδίου *entry*.

Υπάρχουν πολλοί τρόποι να οριστούν τα entry points. Η γενική σύνταξη είναι *entry: {<entryChunckName> string/[string]}*

```
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

```
module.exports = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

```
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js'
  }
};
```

```
module.exports = {
  entry: {
    filename: '[name].bundle.js'
  }
};
```

Το να δοθεί πίνακας από αρχεία είναι χρήσιμο, όταν θέλουμε να χρησιμοποιήσουμε πολλά εξαρτώμενα αρχεία και να καταγράψουμε τις εξαρτήσεις τους σε ένα chunk.

Σε εφαρμογές με πολλές σελίδες, είναι καλή πρακτική να ορίζουμε ένα entry point για κάθε μια σελίδα.

```
module.exports = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

### Output
Δείχνει στο webpack που να αποθηκεύει τα bundles που δημιουργεί. Η default τοποθεσία είναι ο './dist' φάκελος και μπορεί να αλλάξει μέσω του πεδίου *output* στο configuration.

Παρ' όλο που μπορεί να υπάρχουν πολλά entry points, μόνο ένα output ορίζεται και χρειάζεται τουλάχιστον να έχει δηλωμένο το filename.

```
module.exports = {
  output: {
    filename: 'bundle.js',
  }
};
```

Όταν έχουμε πολλαπλά entry points, μπορούμε να χρησιμοποιήσουμε *substitutions* για να σιγουρευτούμε πως κάθε αρχείο θα έχει ξεχωριστό όνομα, π.χ έστω ότι έχομυε δύο entry points, το *app* και το *search*, ορίζοντας *output.filename: [name].js* θα δημιουργηθούν δύο output αρχεία, το './dist/app.js' και το './dist/search.js'.

```
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
};

// writes to disk: ./dist/app.js, ./dist/search.js
```

Τα substitutions είναι τα εξής: 

1. **[hash]** : το hash του module identifier
2. **[contenthash]**  : το hash του κάθε αρχείου
3. **[chunkhash]** : το hash του περιεχομένου του chunk
4. **[name]**  : το όνομα του module
5. **[id]** : το αναγνωριστικό του module
6. **[query]**  : το query του module, δηλ το string που ακολουθεί το ? στο filenae
7. **[function]** : η συνάρτηση που μπορεί να επιστρέφει το filename

### Loaders
Το webpack υποστηρίζει κατ' εξοχήν μόνο js και json αρχεία.

Για να χρησιμοποιηθούν και άλλου είδους αρχεία (π.χ. css), υπάρχουν οι loaders, που τα μετατρέπους σε έγκυρα modules ώστε να χρησιμοποιηθούν στο depedency graph. 

Θα πρέπει πρώτα να εγκατασταθούν μέσω του npm π.χ. ```npm install --save-dev css-loader ts-loader```

Οι loaders μπορούν να οριστούν με τρείς τρόπους:

#### 1.Configuration
Στο πεδίο modules.rules ορίζονται οι loaders.

To rules είναι ένα  array και κάθε στοιχείο του έχει δύο πεδία:
1. **test**, που καθορίζει τί είδους αρχεία θα μετατραπούν 
2. **use**, που καθορίζει ποιός loader θα χρησιμοποιηθεί
Ο πίνακας αυτός διαβάζεται από πάνω προς τα κάτω.

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // style-loader
          { loader: 'style-loader' },
          // css-loader
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          // sass-loader
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

#### 2.Inline
Οι loaders μπορούν να οριστούν, βάζοντάς τους πριν το string της διαδρομής (μέσα σε αυτό) σε ένα import statement, διαχωρίζοντάς τους με !
π.χ. ```import Styles from 'style-loader!css-loader?modules!./styles.css';```

* Βάζοντας στην αρχή !, απενεργοποιούμε όλους τους normal loaders που έχουν οριστεί στο configuration.
π.χ. ```import Styles from '!style-loader!css-loader?modules!./styles.css';```

* Βάζοντας στην αρχή !!, απενεργοποιούμε όλους τους loaders, preLoaders, postLoaders που έχουν οριστεί στο configuration.
π.χ. ```import Styles from '!!style-loader!css-loader?modules!./styles.css';```

* Βάζοντας στην αρχή -!, απενεργοποιούμε όλους τους loaders και τους preLoaders, αλλά όχι τους postLoaders που έχουν οριστεί στο configuration.
π.χ. ```import Styles from '-!style-loader!css-loader?modules!./styles.css';```

#### 3. CLI
Οι loaders μπορούν να οριστούν μέσω του CLI, με το option --module -bind <loaders>, χρησιμοποιώντας με το ίδιο τρόπο όπως πριν το ! για τον διαχωρισμό τους, αν χρειαστεί.

```
webpack --module-bind pug-loader --module-bind 'css=style-loader!css-loader'
```

Ιδιότητες των loaders:

1. Μπορούν να γίνουν chain. Κάθε loader έχει εφαρμογή στο ήδη επεξεργασμένο αποτέλεσμα και εκτελούνται με ανάποδη σειρά.
2. Μπορούν να είναι synchronous και asynchronous.
3. Τρέχουν πάνω στο node και άρα μπρούν να χρησιμοποιήσουν όλες του τις δυνατότητες.
4. Μπορούν να παραμετροποιηθούν με το αντικείμενο options.
5. Τα κανονικά modules, μπορούν να κάνουν export έναν loader μέσω του package.json και του πεδίου loader.
6. Τα plugins μπορούν να επεκτείνουν την λειτουργικότητά τους.
7. Μπορούν να αποθέσουν επιπρόσθετα, δευτερεύοντα αρχεία.

### Plugins 
Χρησιμοποιούνται για την επέκταση των δυνατοτήτων του webpack (π.χ. optimization, asset management, environment variables injection).

Για να χρησιμοποιηθεί ένα plugin, πρέπει να γίνει ```require()``` στην αρχή και να προστεθεί στον πίνακα *plugins* του configuration, δημιουργώντας ένα νεό instance με το ```new```.

Ουσιαστικά, τα plugins κάνουν οτιδήποτε δεν μπορούν να κάνουν οι loaders.

Είναι αντικείμενα που έχουν την μέθοδο ```apply(comiler)```, η οποία εκτελείτα από το webpack και δίνει στον προγραμματιστή έλεγχο πάνω σε όλη την διαδικασία του compilation.

Μπορούν να χρησιμοποιηθούν με τους εξής τρόπους:

#### 1.Configuration
Δημιουργώντας instances μέσα στον πίνακα plugins του configuration.

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

#### 2.Node API
Μέσω του API του Node js.

```
const webpack = require('webpack'); //to access webpack runtime
const configuration = require('./webpack.config.js');

let compiler = webpack(configuration);

new webpack.ProgressPlugin().apply(compiler);

compiler.run(function(err, stats) {
  // ...
});
```


### Configuration
Είναι αρχείο γραμμένο σε Node js, άρα μπορεί να χρησιμοποιήσει όλες του τις δυνατότητες.

Θα πρέπει **να αποφεύγονται** οι εξής πρακτικές:

1. Χρήση παραμέτρων του CLI, όταν χρησιμοποιούμε το webpack CLI (προτείνεται η δημιουργία δικού μας CLI ή η χρήση του *--env*).
2. Export μη απόλυτων, περιγραφικών τιμών.
3. Μεγάλα configuration αρχεία (συνίσταται το σπάσιμο σε μικρότερα αρχέια).

```
var path = require('path');

module.exports = {
  mode: 'development',
  entry: './foo.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'foo.bundle.js'
  }
};
```


### Modules
To webpack χρησιμοποιεί τα modules με τον ίδιο τρόπο που χρησιμοποιούνται και γενικά στον προγραμματισμό.

Πέραν αυτού, τα modules του webpack μπορούν αν εκφράσουν τα depedencies με πολλούς τρόπους:

1. ```import()```
2. ```require()```
3. AMD define και require statement
4. ```@import``` μέσα σε css/sass/less αρχεία
5. url εικόνας μέσα σε stylesheet ή HTML αρχεία

Επίσης το webpack μπορεί να χρησιμοποιήσει modules σε πολλές γλώσσες προγραμματισμού, με την χρήση των κατάλληλων loaders.

Μερικές από αυτές είναι: 

* Coffee Script
* Type Scrtipt
* ESNext (Babel)
* Sass
* Less
* Stylus
* Elm

### Module resolution
Το webpack χρησιμοποιεί την βιβλιοθήκη *enhance-resolve* για να ανακτά τις διαδρομές των αρχείων.

Η ανάκτηση μπορεί να αναγνωρίσει τρεις μορφές διαδρομών: 

#### 1. Absolute paths 
π.χ. ```import '/home/me/file';```
    ``` import 'C:\\Users\\me\file';```

#### 2. Relative paths
π.χ. ```import '../src/file1';```
    ```import './file2';```

#### 3. Module paths
π.χ. ```import 'module';```

Σε αυτήν την περίπτωση γίνεται αναζήτηση των ζητούμενων modules, στους φακέλους που ορίζονται στο πεδίο *resolve.modules* του configuration.

Η κατ' εξοχήν διαδρομή των modules μπορεί να αλλάξει δημιουργώντας ένα alias μέσω του πεδίου *resolve.alias* του configuration.

* Αν η ζητούμενη διαδρομή δείχνει σε αρχείο, τότε αυτό αν έχει επέκταση γίνειτα bundle κατευθείαν, ειδάλλως η επέκταση ορίζεται μέσω της τιμής του πεδίου *resolve.extensions* του  configuration.

* Αν δείχνει σε φάκελο, τότε εαν αυτός περιέχει *package.json*, αναζητούνται σε αυτόν με σειρά τα πεδία που ορίζονται στο πεδίο *resolve.mainFields* του configuration και το πρώτο που θα βρεθεί ορίζει την διαδρομή του module.
Εάν δεν περιέχει *package.json* ή η αναζήτηση των πεδίων του *resolve.mainFields* δεν επιστρέφει κάτι, τότε αναζητώνται στον φάκελο με σειρά τα αρχεία που ορίζονται στο πεδίο *resolve.mainFiles* του configuration. Η επέκταση των αρχείων σε κάθε περίπτωση, εάν δεν υπάρχει, ορίζεται με τον τρόπο που αναφέρθηκε προηγουμένως.

Για τους loaders ακολουθούνται οι ίδιες διαδικασίες, αλλά μπορούν να οριστούν διαφορετικά rules, μέσω του πεδίου *resolve.loaders* του configuration.

Όσον αφορά το **caching**, κάθε παρέμβαση στο filesystem γίνεται cache. Αν το *watch mode* είναι ενεργοποιημένο, τότε μόνο τα τροποποιημένα αρχεία αφαιρούνται από την cache, ενω αν είναι απενεργοποιημένο, αυτή καθαρίζεται πριν κάθε compilation.

### Targets
Ορίζουν το περιβάλλον αναφοράς της εφαρμογής, καθώς η js μπορεί να έχει γραφεί είτε για server, είτε για browser.

```
module.exports = {
  target: 'node'
};
```

Δεν υπάρχει η δυνατότητα για πολλαπλές τιμές στο πεδίο *target*.

Αν αυτό χρειαστεί θα πρέπει να δημιουργηθούν πολλαπλά παρόμοια bundles και configurations.

```
const path = require('path');
const serverConfig = {
  target: 'node',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.node.js'
  }
  //…
};

const clientConfig = {
  target: 'web', // <=== can be omitted as default is 'web'
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.js'
  }
  //…
};

module.exports = [ serverConfig, clientConfig ];
```


### The manifest
Είναι η συλλογή δεδομένων σχετικά με τα modules, που βοηθούν στην ανάκτηση, την φόρτωση, το caching και το bundling των modules.

### Hot Module Replacemnt (HMR)
Είναι η τροποποίηση modules, ενόσω η εφαρμογή τρέχει, χωρίς full reload. 

Αυτό επηρεάζει τους εξής τομείς: 

#### 1. Την εφαρμογή
Μπορεί η εν λόγω διαδικασία να οριστεί, ώστε να γίνεται αυτόματα, αλλά υπάρχει και η δυνατότητα να απαιτείτα διάδραση του χρήστη για να γίνουν οι αλλαγές.

#### 2. Τον compiler
Θα πρέπει να διενεργηθεί και μια ενημέρωση στον compiler, προκειμένου να γίνει η μετάβαση από μαι προηγούμενη στην νεά έκδοση.

Αυτή η ενημέρωση απότελείται από δύο μέρη: 

1. Το ενημερωμένο manifest, που περιέχει το νέο hash και τα ενημερωμένα chunks.
2. Το ενημερωνένο module.

#### 3. Τα modules
Όταν υλοποιείται το HMR Interface σε ένα module, μπορεί να οριστεί το τι θα συμβεί όταν θα ενημερωθεί το module.

#### 4. Το runtime 
Το runtime υποστηρίζει δύο μεθόδους: ```ckeck``` και ```apply```.

* Η *check* κάνει ένα HTTP request στο ανανεωμένο manifest. Αν το request αποτύχει, σημαίνει πως δεν υπάρχει update. Αν πετύχει, η λίστα των ανανεωμένων chunks συγκρίνεται με την ήδη υπάρχουσα. Για κάθε φορτωμένο chunk, γίνεται λήψη του ανανεωμένου. Όταν ληφθούν όλα και είναι έτοιμα για εφαρμογή, το runtime αλλάζει σε κατάσταση *ready*.

*  Η *apply* σηματοδοτεί όλα τα αποθηκευμένα modules σαν εσφαλμένα. Για κάθε εσφαλμένο module, θα πρέπει να υπάρχει ένα *update handles* στο ίδιο ή στα parent modules του. Σε άλλη περίπτωση η σηματοδότηση ανεβαίνει στο parent module κ.ο.κ. μέχρι να φτάσει στον entry point, ή σε αρχείο με update handler. Αν γίνει το πρώτο, η διαδικασία αποτυγχάνει. Μετά από αυτό, όλα τα εσφαλμένα modules αφαιρούνται. Τότε το τρέχον hash ενημερώνεται και όλλοι οι *accept handlers* καλούνται. Το runtime αλλάζει κατάσταση σε *idle* και η εκτέλεση συνεχίζει κανονικά.

#### API
Το αναλυτικό api του webpack, καθώς και όλα τα options για το configuration μπορούν αν βρεθούν [εδώ.](https://webpack.js.org/configuration/)


**Όλες οι πληροφορίες του παρόντος οδηγού, αντλήθηκαν από το official website του webpack, που μπορεί να βρεθεί [εδώ.](https://webpack.js.org/)**




