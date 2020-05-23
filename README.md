# cv19

Setup for ReactJS app with Cordova:

REFERENCE: https://medium.com/@pshubham/using-react-with-cordova-f235de698cc3

Forked our Cordova Repo to https://github.com/goobito/cv19-app-cordova-react.git
ALTERNATELY CREATE NEW CORDOV PROJECT (see earlier) I chose to fork so I can later pull this work back into the cv19 client app project to see the exact changes).

> git clone https://github.com/goobito/cv19-app-cordova-react.git
> cd cv19-app-cordova-react

Test our forked project runs OK:

> cordova run android

Gives error need to install android. This is because our REPO does not store the platforms, only the core files. This is because we ant to develop OONE code base to thn RUN on dfiferent platforms. So we only REPO the core.

So, add the android platform (adds folder to project: platforms/android);

> cordova platform add android

Then stare an emulator::

> emulator -avd Galaxy_Nexus_API_28

Then:

> cordova run android

Cool. All works fine. Now we can start to bring in React:

CREATE REACT APP:
Exit our project (up one folder)
Create a separate project using npx create-react-app {myappname}
NOTE: Assumes node, npm installed.

> npx create-react-app cv19-react-template
> Test install OK:
> cd cv19-react-template
> npm start
> Verify at http://localhost:3000/

OK- we now have a React JS App.

Next we need to merge the react App into the Cordova Project:

MERGE REACT INTO CORDOVA PROJECT:
COPY: src and public folders from React to our Cordova project):

> cp -r src ../cv19-app-react/
> cp -r public ../cv19-app-react/

MERGE: package.json from react project into cordova project:
“scripts”,
“dependencies”
“Browserslist”
No load all dev depebdencies in our cordova projact:

> cd cv19-app-cordova-react
> npm install
> Verify OK:
> npm start

Verify at http://localhost:3000/

At this point we have our Cordva App and our react in the same project and caan run either. But we do not have the Reat App loading inside cordova. Thats our next task.

INDEX.HTML: CONFIGURE CORDOVA in public/index.html

Add to <HEAD> section:

`

<meta http-equiv="Content-Security-Policy"
     content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; media-src *; img-src 'self' data: content:;"
   />
   <meta name="format-detection" content="telephone=no" />
   <meta name="msapplication-tap-highlight" content="no" />
   <meta
     name="viewport"
     content="initial-scale=1, width=device-width, viewport-fit=cover"
   />
`
Load Cordova, before `</BODY>`:

<pre>
   <script src="cordova.js" type="text/javascript"></script>
</pre>

INDEX.JS Load react after device ready:

<pre>
const renderReactDom = () => {
 ReactDOM.render(<App />, document.getElementById("root"));
};
 
if (window.cordova) {
 document.addEventListener(
   "deviceready",
   () => {
     renderReactDom();
   },
   false
 );
} else {
 renderReactDom();
}
</pre>

RELATIVE PATHS FOR FILE LOADING (add to package..json):
"homepage": "./"

Add PRE-BUILD Cordova Hook (to config.xml). Necessary to run React Build then make built resources (in build folder) available in www folder.

We need to build a script that will move/rename folders.

Use rimraf for The UNIX command rm -rf for node.

> npm i rimraf --save

Create /scripts/prebuild.js to rename folder build to www:

<pre>
const path = require('path');
const { exec } = require('child_process');
const fs = require('fs');
const rimraf = require('rimraf');

function renameOutputFolder(buildFolderPath, outputFolderPath) {
    return new Promise((resolve, reject) => {
        fs.rename(buildFolderPath, outputFolderPath, (err) => {
            if (err) {
                reject(err);
            } else {
                resolve('Successfully built!');
            }
        });
    });
}

function execPostReactBuild(buildFolderPath, outputFolderPath) {
    return new Promise((resolve, reject) => {
        if (fs.existsSync(buildFolderPath)) {
            if (fs.existsSync(outputFolderPath)) {
                rimraf(outputFolderPath, (err) => {
                    if (err) {
                        reject(err);
                        return;
                    }
                    renameOutputFolder(buildFolderPath, outputFolderPath)
                        .then(val => resolve(val))
                        .catch(e => reject(e));
                });
            } else {
                renameOutputFolder(buildFolderPath, outputFolderPath)
                    .then(val => resolve(val))
                    .catch(e => reject(e));
            }
        } else {
            reject(new Error('build folder does not exist'));
        }
    });
}

module.exports = () => {
    const projectPath = path.resolve(process.cwd(), './node_modules/.bin/react-scripts');
    return new Promise((resolve, reject) => {
        exec(`${projectPath} build`,
            (error) => {
                if (error) {
                    console.error(error);
                    reject(error);
                    return;
                }
                execPostReactBuild(path.resolve(__dirname, '../build/'), path.join(__dirname, '../www/'))
                    .then((s) => {
                        console.log(s);
                        resolve(s);
                    })
                    .catch((e) => {
                        console.error(e);
                        reject(e);
                    });
            });
    });
};
</pre>

SRC:https://gist.githubusercontent.com/pshubham95/fa78e35d328215f3206d6829fe8d4604/raw/976fbbb3f1f455977c978e26df7dd3eee804f34a/prebuild.js

Test: Start Emultor and run Cordova:

> emulator -avd Galaxy_Nexus_API_28
> cordova run android

Yay!We have React deployed and running OK on Cordova.

Lastly, we need an empty www folder, otherwise "cordova run" will fail saying "Not a Cordova App"

To run, execute:
(to build React into /build)

> npm run build
> , then
> (prehook copies React from build)
> cordova run android

BEFORE: created a second pre build hook to pre run "react build" and copy to www folder
