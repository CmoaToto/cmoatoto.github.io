---
layout: post
title:  "MiraSlide : Développer une application Android (4.2+) multi-écran exploitant le WiDi par l'exemple"
date:   2013-12-14 12:00:00
tags: android miracast application french tutorial
---
Dans ce billet, je vais vous présenter les différentes étapes pour développer une application Android exploitant le WiDi. A l'issue de cette lecture, vous devriez connaitre les bases pour développer vos propres solutions multi-écrans sous Android. Cependant, le WiDi est loin d'être une solution cantonnée au monde Android et je vous invite à lire l'article de [Pierre S](http://software.intel.com/fr-fr/user/991216) sur le [développement d'application WinJS (Windows 8.1)](http://software.intel.com/fr-fr/blogs/2013/12/10/d-velopper-une-application-winjs-20-windows-81-multi-crans-avec-widi).

- [Prérequis][1]
- [MiraSlide : Une application de Présentations][2]
- [Utiliser le WiDi dans Android][3]
- [Implémentation du code dans MiraSlide][4]
- [Conclusion][5]
- [Sources][6]

    [1]: #prerequis
    [2]: #intro-miraslide
    [3]: #widi-dans-android
    [4]: #implem-miraslide
    [5]: #conclusion
    [6]: #sources

Avant tout, il va falloir se familiariser avec certaines connaissances.

<a name="prerequis" />

Prérequis
=========

**Android :** Si vous débutez sur le développement Android, je vous conseille très fortement de lire tous les guides de démarrage sur [le site des développeurs Android](http://developer.android.com/index.html).

**Wireless Display (ou WiDi) :** Cette technologie de diffusion de flux audio et vidéo développée par Intel est une alternative aux technologies Miracast du consortium Wi-Fi Alliance, et AirPlay d'Apple. Cependant, la technologie WiDi permet aux appareils le supportant de communiquer avec les récepteurs Miracast. Finalement le WiDi se veut être une "super-surcouche" à Miracast. Afin d'en apprendre plus sur le WiDi, je vous conseille de lire cette présentation par [Pierre S](http://software.intel.com/fr-fr/user/991216) : [Tout savoir sur le WiDi](http://software.intel.com/fr-fr/blogs/2013/12/09/tout-savoir-sur-le-widi).

Enfin, si vous voulez pousser encore plus loin vos connaissances sur le Wireless Display pour Android, vous pouvez consulter la vidéo de la conférence au Paris Android User Group sur la [présentation du WiDi pour Android](http://www.youtube.com/watch?v=gZopNFkeHH4) par [Xavier Hallade](http://ph0b.com/) ([slides ici](http://fr.slideshare.net/bemyapp/wi-di-android-applications-development))

Assez de liens, passons au développement !

<a name="intro-miraslide" />

MiraSlide : Une application de Présentations
============================================

![MiraSlide](/assets/images/miraslide_logo.png "MiraSlide logo")

Code source du projet MiraSlide

Page Google Play de l'application

Le but de cette application est de répondre à un besoin simple : les conférenciers sont soit encombrés, soit esclaves du matériel prêté, soit les deux. Ici, le Wireless Display associé à un appareil plus léger qu'un ordinateur et pouvant donc être utilisé en télécommande nous permettrait de diffuser et commander les slides depuis son téléphone/tablette, en ayant en plus dans la main les informations supplémentaires que nous apporte un ordinateur (chronomètre, notes...).

Voici donc le but très simple de MiraSlide :

Vous sélectionnez votre fichier de présentation, vous connectez votre appareil à un écran récepteur Wireless Display puis vous lancez la présentation.

Sur l'écran récepteur, la première page de votre présentation s'affiche.

L'écran de votre appareil vous propose alors un chronomètre, ainsi qu'une télécommande affichant le slide courant, les notes éventuelles, et des boutons précédent / suivant.

MiraSlide presentation

Maintenant que nous avons une idée plus précise de l'application que nous voulons développer, nous allons nous pencher sur les APIs.

Utiliser le WiDi dans Android

Pour ceux qui n'auraient pas bien lu les prérequis, le WiDi (et plus largement la notion d'écran externe) apparaît dans le framework d'Android avec l'API 17 (Android 4.2.2). Deux éléments essentiels sont alors créés :

Le Display Manager va être l'interface permettant à l'application de connaitre les écrans disponibles et d’interagir avec.
Une Presentation est une vue similaire à une Dialog (dont elle est étendue) mais projetée sur un Display donné. Une des notions les plus importantes à comprendre du fait que la Presentation est une extension d'une Dialog est qu'elle est forcément attachée à une Activity. Ainsi, si cette dernière est mise en pause (si elle n'est plus visible à l'écran, en gros), alors la Presentation n’apparaîtra plus sur le Display associé (et le mode d'écran clone par défaut s'activera). Si vous retournez ensuite à votre Activity, la Presentation reviendra s'afficher sur le Display.
L'implémentation est finalement très simple :

La récupération du Display
La création et l'affichage de la Presentation
Ajout de listeners
1. La récupération du Display
Elle peut se faire de deux façons. Ou bien en utilisant le MediaRouter introduit avec l'API 16 :

01
MediaRouter mediaRouter = (MediaRouter) context.getSystemService(Context.MEDIA_ROUTER_SERVICE);
02
MediaRouter.RouteInfo route = mediaRouter.getSelectedRoute();
03
 
04
if (route != null) {
05
    Display presentationDisplay = route.getPresentationDisplay();
06
 
07
    if (presentationDisplay != null) {
08
 
09
        // Your code...
10
 
11
    }
12
}
Ou bien en utilisant le Le Display Manager :

01
DisplayManager displayManager = (DisplayManager) mActivity.getSystemService(Context.DISPLAY_SERVICE);
02
 
03
// Selecting DISPLAY_CATEGORY_PRESENTATION prevents the DisplayManager from returning inapropriate Display,
04
// like the own device display.
05
Display[] displays = displayManager.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION);
06
 
07
if (displays.length == 0) {
08
 
09
    // If there is no external display connected, we launch the Display settings. We could launch
10
    // the Wifi display settings with ACTION_WIFI_DISPLAY_SETTINGS but it is a hidden static value
11
    // because there may be not such settings (if the device does not have Wireless display but
12
    // have API >= 17).
13
    startActivity(new Intent(Settings.ACTION_DISPLAY_SETTINGS));
14
 
15
} else {
16
 
17
    // We should show a DialogBox to let the user select the display if there is more than one but
18
    // for this example we only choose the first one
19
    Display display = displays[0];
20
 
21
}
2. La création et l'affichage de la Presentation
Ici non plus, rien de très compliqué. La Presentation n'a besoin que de l'Activity parente et du Display où être affiché pour être créé. Ensuite, la méthode show() affiche la Presentation sur le Display. Comme une Dialog, la Presentation comprend une méthode setContentView() grâce à laquelle vous pourrez définir la vue à afficher :

01
private void showPresentation() {
02
    mPresentation = new MyPresentation(this, mDisplay);
03
    mPresentation.show();
04
}
05
 
06
private class MyPresentation extends Presentation {
07
 
08
    /* constructors ... */
09
 
10
    @Override
11
    public void onCreate(Bundle savedInstanceState) {
12
        super.onCreate(savedInstanceState);
13
        View v = getLayoutInflater().inflate(R.layout.presentation, null);
14
        setContentView(v);
15
    }
16
}
3. Ajout de listeners
Pour rendre votre système plus fiable, vous pouvez rajouter des listeners au DisplayManager afin d'être prévenu lorsque des Display sont ajoutés ou retirés. Ceci est particulièrement pratique pour éviter à une Presentation de tenter de continuer à fonctionner alors que le Display qui lui est associé a été déconnecté :

01
mDisplayManager.registerDisplayListener(new DisplayListener() {
02
 
03
    @Override
04
    public void onDisplayRemoved(int displayId) {
05
        // Stop presentation ...
06
        // Show a message to the user to reconnect the display
07
 
08
    }
09
 
10
    @Override
11
    public void onDisplayChanged(int displayId) {
12
        // Something happend. You should check if everything is ok before continuing
13
 
14
    }
15
 
16
    @Override
17
    public void onDisplayAdded(int displayId) {
18
        // If you were waiting for a display, maybe you should use it !
19
 
20
    }
21
}, null);
Implémentation du code dans MiraSlide

Nous n'allons bien évidemment pas revenir sur tout le code de MiraSlide, ce serait long et inutile, car finalement le code concernant le Wireless Display est assez court face au reste du code. Nous allons donc nous focaliser sur les points suivants :

La récupération du Display
La sélection du Display
La création et l'affichage de la Presentation
Le contrôle de la Presentation
Cependant, je vais rapidement expliquer la structure générale du code qui se divise en 4 éléments principaux.

la MainActivity est l'unique Activity de l'application. Ainsi, si la Presentation est en cours et que l'on se déplace entre les différentes vues, la Presentation ne s'arrête pas.
le SelectionFragment est la première vue, qui va permettre à l'utilisateur de sélectionner le fichier (PDF) contenant les slides, ainsi que le Display sur lequel afficher la Presentation. Enfin, il permet de lancer la-dite Presentation.
le ControllerFragment est la vue s'affichant lorsque l'on lance la Presentation. Elle comprend un chronomètre, le slide courant ainsi que des boutons précédent et suivant.
le PdfViewerPresentation est la vue Presentation gérant ce qui est affiché sur le Display. Il comprend le moteur permettant de récupérer et d'afficher les images des slides demandés par le ControllerFragment.
1. La récupération du Display
À la création du SelectionFragment, on récupère le DisplayManager et on déclare les listeners.
Chaque fois que le SelectionFragment est lancé ou relancé (dans le onResume), on va vérifier l'état des Displays. Si il n'yen a aucun, on propose d'afficher les paramètres d'affichages, si il y en a un seul, on le sélectionne automatiquement, et s'il yen a plus, on propose de sélectionner le Display voulu
Voici le code associé :

01
public class SelectionFragment extends Fragment implements OnClickListener, DisplayListener {
02
 
03
    // ...
04
 
05
    // The display manager is the object to get information about the different displays
06
    private DisplayManager mDisplayManager;
07
 
08
    @Override
09
    public void onCreate(Bundle savedInstanceState) {
10
        super.onCreate(savedInstanceState);
11
 
12
        // ...
13
 
14
        // We get the display manager to get info about the display. We also register to any change
15
        // about the (dis)connection of the displays.
16
        mDisplayManager = (DisplayManager) mActivity.getSystemService(Context.DISPLAY_SERVICE);
17
        mDisplayManager.registerDisplayListener(this, null);
18
    }
19
    
20
    @Override
21
    public void onResume() {
22
        super.onResume();
23
 
24
        // On resume, we check the state of each buttons.
25
        checkLaunchable(getView());
26
    }
27
 
28
    // We check the selection of the pdf file and the display, and we update the color of the buttons and we update
29
    // if the "Launch Projection" button should be enabled
30
    // @param v : the global view of the fragment
31
    private void checkLaunchable(View v) {
32
        checkDisplay((TextView) v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay));
33
 
34
        if (mActivity.getDislay() != null) {
35
            v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay).setBackgroundResource(R.drawable.button_green);
36
        } else {
37
            v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay).setBackgroundResource(R.drawable.button_red);
38
        }
39
 
40
        // ...
41
 
42
    }
43
 
44
    // We check the state of the external displays, update the text of the display button, and if there is only
45
    // one external display, we auto select it
46
    private void checkDisplay(TextView displayButton) {
47
        Display[] displays = mDisplayManager.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION);
48
 
49
        if (displays.length > 1 && mActivity.getDislay() == null) {
50
            displayButton.setText("Select a wireless display");
51
        } else if (displays.length == 1) {
52
            mActivity.setDisplay(displays[0]);
53
            displayButton.setText("Display selected " + displays[0].getName());
54
        } else {
55
            mActivity.setDisplay(null);
56
            displayButton.setText("Connect to a wireless display");
57
        }
58
    }
59
 
60
    // Methods called when a display is added or removed. We change the button state if we add or remove a
61
    // display, and we stop the presentation if there is a display removed.
62
 
63
    @Override
64
    public void onDisplayAdded(int displayId) {
65
        checkLaunchable(getView());
66
    }
67
 
68
    @Override
69
    public void onDisplayChanged(int displayId) {
70
    }
71
 
72
    @Override
73
    public void onDisplayRemoved(int displayId) {
74
        if (mActivity.getDislay() != null && displayId == mActivity.getDislay().getDisplayId()) {
75
            mActivity.stopPresentation();
76
            checkLaunchable(getView());
77
        }
78
    }
79
}
2. La sélection du Display
Comme décrit plus haut, le bouton de sélection du Display va évoluer en fonction des Display connectés à l'appareil :

Si aucun Display n'est connecté, le bouton affiche Connect to a wireless display. Si l'utilisateur clique dessus, le code suivant est appelé. La fenêtre des paramètres d'affichage du téléphone est alors affiché. Si l'appareil est compatible Wireless Display, un bouton Screen mirroring, ou Wireless Display ou une traduction devrait apparaitre (cf image ci dessous). En cliquant dessus, la liste des appareils visibles compatible Miracast apparait. L'utilisateur n'a plus qu'à cliquer dessus pour s'y connecter.

1
if (displays.length == 0) {
2
    // If there is no external display connected, we launch the Display settings. We could launch the Wifi display
3
    // settings with ACTION_WIFI_DISPLAY_SETTINGS but it is a hidden static value because there may be not such settings
4
    // (if the device does not have Wireless display but have API >= 17).
5
    startActivity(new Intent(Settings.ACTION_DISPLAY_SETTINGS));
6
}
Connecting a Wireless Display on Android

Si un seul Display est connecté, ou si un Display a déjà été sélectionné, le bouton affiche Display selected NOM_DU_DISPLAY. Si l'utilisateur clique dessus, le même code que lors du cas où plusieurs Displays sont connectés est executé.

Si plusieurs Displays sont connectés, le bouton affiche Select a wireless display. Si l'utilisateur clique dessus, le code suivant est appelé. Une Dialog box s'ouvre, listant la liste des Displays disponibles. Si l'utilisateur clique sur un des Displays, ce dernier est alors sélectionné et on se retrouve dans le cas précédent.

01
// If there is one or more external display, we show a dialog box with the list of the display.
02
// The user can select the display he wants, or close the dialog.
03
final ArrayAdapter arrayAdapter = new ArrayAdapter(mActivity, android.R.layout.select_dialog_singlechoice);
04
for (int i = 0; i < displays.length; i++) {
05
    arrayAdapter.add(displays[i].getName());
06
}
07
 
08
 AlertDialog.Builder builder = new AlertDialog.Builder(mActivity).setIcon(R.drawable.ic_launcher).setTitle("Select a display")
09
        .setNegativeButton("cancel", null).setAdapter(arrayAdapter, new DialogInterface.OnClickListener() { 
10
 
11
            @Override
12
            public void onClick(DialogInterface dialog, int which) {
13
                // When the user choose a display through the dialog, we set it in the parent Activity and update
14
                // the state of the buttons.
15
                mActivity.setDisplay(displays[which]);
16
                checkLaunchable(getView());
17
            }
18
});
19
builder.show();
3. La création et l'affichage de la Presentation
Lorsque le Display est sélectionné, ainsi qu'un fichier PDF, le bouton Launch Projection est activé. Lorsque l'utilisateur clique dessus, la Presentation est créée et affichée. On bascule alors l'utilisateur sur la vue Controller. Voici le code exécuté :

01
// SelectionFragment.java
02
 
03
// ...
04
 
05
@Override
06
public void onClick(View v) {
07
    if (v.getId() == R.id.fragment_selection_button_launchprojection) {
08
        // On click on the "launch projection" button, we... launch the projection
09
        mActivity.launchPresentation();
10
 
11
}
01
// MainActivity.java
02
 
03
// ...
04
 
05
// Set the presentation (created in the selection fragment). This method creates listeners to control the visibility of the controller fragment and if the
06
// mShowHideControllerActionBarButton should be enabled. Then it shows the presentation.
07
//
08
// @param presentation : the presentation to show
09
//
10
public void launchPresentation() {
11
    mPresentation = new PdfViewerPresentation(this, mDisplay, mPdfPath);
12
 
13
    mPresentation.setOnShowListener(new OnShowListener() {
14
 
15
        @Override
16
        public void onShow(DialogInterface dialog) {
17
 
18
            showHideController(true);
19
            enableShowHideControllerActionBarButton(true);
20
            mControllerFragment.notifyViewPager();
21
        }
22
    });
23
    mPresentation.setOnDismissListener(new OnDismissListener() {
24
 
25
        @Override
26
        public void onDismiss(DialogInterface dialog) {
27
            showHideController(false);
28
            enableShowHideControllerActionBarButton(false);
29
        }
30
    });
31
 
32
    mPresentation.show();
33
}
L'initialisation de la Presentation est très simple mais le code peut paraître un peu compliqué. Ceci est dû à la préparation du PDF et de son affichage. Ce qu'il faut retenir est que l'initialisation se fait dans le constructeur, et que la création de la vue à afficher, et l'affichage du premier slide, se fait dans le OnCreate(Bundle), et est appliqué grâce à la méthode setContentView(View). Voici le code très simplifié.

01
// PdfViewerPresentation.java
02
 
03
// ...
04
 
05
// Constructor. It creates the Presentation, then load the display info and the Pdf to show
06
public PdfViewerPresentation(Context context, Display display, String pdfFilePath) {
07
    super(context, display);
08
    mPdfFilePath = pdfFilePath;
09
 
10
    // ...
11
}
12
 
13
@Override
14
public void onCreate(Bundle savedInstanceState) {
15
    super.onCreate(savedInstanceState);
16
 
17
    createContentView();
18
 
19
    showPage();
20
}
21
 
22
// Create the imageView to show the pdf pages Bitmaps
23
private void createContentView() {
24
    mImageView = (ImageView) getLayoutInflater().inflate(R.layout.presentation_main, null);
25
    setContentView(mImageView);
26
}
27
 
28
// Show the current page on the Presentation view
29
private void showPage() {
30
    mImageView.setImageBitmap(getPage(mPage));
31
}
32
 
33
// return the Bitmap of the pdf specified page
34
public Bitmap getPage(int page) {
35
     
36
    // ...
37
}
4. Le contrôle de la Presentation
Une fois la Presentation lancée, l'utilisateur peut la contrôler grâce au ControllerFragment. Ce dernier comprend un ViewPager qui affiche les slides du PDF. Lorsque l'utilisateur change de slide (soit en faisant glisser, soit en appuyant sur les boutons Précédent et Suivant), le ControllerFragment prévient l'Activity parente du nouveau slide sélectionné, et l'Activity fait remonter l'information à la Presentation afin qu'elle se mette à jour. Voici le code correspondant :

01
// ControllerFragment.java
02
 
03
// ...
04
 
05
// We create the viewpager which shows the pages of the pdf file. If the user changes the slide, the listeners
06
// updates tell the parent Activity to change the image in the presentation.
07
//
08
// @param v : the fragment view
09
//
10
private void createViewPager(View v) {
11
    mSlidesPagerAdapter = new SlidesPagerAdapter(((FragmentActivity) getActivity()).getSupportFragmentManager());
12
    mSlidesViewPager = (ViewPager) v.findViewById(R.id.fragment_controller_pager);
13
    mSlidesViewPager.setAdapter(mSlidesPagerAdapter);
14
    mSlidesViewPager.setOnPageChangeListener(new SimpleOnPageChangeListener() {
15
 
16
        @Override
17
        public void onPageSelected(int page) {
18
            ((MainActivity) getActivity()).getPresentation().moveTo(page);
19
        }
20
    });
21
}
22
 
23
@Override
24
public void onClick(View v) {
25
if (v.getId() == R.id.fragment_controller_button_pageprev) {
26
    // On click on the prev button, we move the viewpager one slide back (which will move the presentation slide as well)
27
    mSlidesViewPager.setCurrentItem(mSlidesViewPager.getCurrentItem() - 1);
28
 
29
} else if (v.getId() == R.id.fragment_controller_button_pagenext) {
30
    // On click on the next button, we move the viewpager one slide next (which will move the presentation slide as well)
31
    mSlidesViewPager.setCurrentItem(mSlidesViewPager.getCurrentItem() + 1);
32
}
1
// MainActivity.java
2
 
3
// ...
4
 
5
// return the Presentation if it has been created. A presentation needs a display and a pdf file
6
public PdfViewerPresentation getPresentation() {
7
    return mPresentation;
8
}
01
// PdfViewerPresentation.java
02
 
03
// ...
04
 
05
// Move the presentation to the page 'page'
06
public void moveTo(int page) {
07
    if (page >= 0 && page < getPageCount()) {
08
        mPage = page;
09
        showPage();
10
    }
11
}
12
 
13
// Show the current page on the Presentation view
14
private void showPage() {
15
    mImageView.setImageBitmap(getPage(mPage));
16
}
17
 
18
// return the Bitmap of the pdf specified page
19
public Bitmap getPage(int page) {
20
     
21
    // ...
22
}
Conclusion

Développer une application exploitant les écrans externes n'est vraiment pas compliqué sous Android. Le framework est simple et fonctionne bien. Cependant, l'application développée, malgré son potentiel, est très loin de repousser le WiDi dans ses retranchements, comme pourrait le faire une application de jeu 3D, ou de streaming vidéo. Enfin, il existe une partie de l'API sur le Wireless Display qui est actuellement cachée dans le code Android et non disponible dans le SDK. Cette API permet de contrôler soi-même la découverte et la connexion aux écrans externes. L'exploitation de cette API est donc dangereuse car elle peut évoluer sans prévenir ou bien ne pas fonctionner comme espéré d'un appareil à un autre. Cependant, si cela est bien fait, l'exploitation de cette API peut simplifier grandement l'utilisation de l'application par l'utilisateur.

Sources

présentation du WiDi pour Android par Xavier Hallade (slides)

le site de développement Android

Code source du projet MiraSlide

Page Google Play de l'application
