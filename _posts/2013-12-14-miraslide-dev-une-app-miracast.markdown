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

    [1]: #prerequis-1
    [2]: #intro-miraslide-2
    [3]: #widi-dans-android-3
    [4]: #implem-miraslide-4
    [5]: #conclusion-5
    [6]: #sources-6

Avant tout, il va falloir se familiariser avec certaines connaissances.

<a name="prerequis-1" />

Prérequis
---------

**Android :** Si vous débutez sur le développement Android, je vous conseille très fortement de lire tous les guides de démarrage sur [le site des développeurs Android](http://developer.android.com/index.html).

**Wireless Display (ou WiDi) :** Cette technologie de diffusion de flux audio et vidéo développée par Intel est une alternative aux technologies Miracast du consortium Wi-Fi Alliance, et AirPlay d'Apple. Cependant, la technologie WiDi permet aux appareils le supportant de communiquer avec les récepteurs Miracast. Finalement le WiDi se veut être une "super-surcouche" à Miracast. Afin d'en apprendre plus sur le WiDi, je vous conseille de lire cette présentation par [Pierre S](http://software.intel.com/fr-fr/user/991216) : [Tout savoir sur le WiDi](http://software.intel.com/fr-fr/blogs/2013/12/09/tout-savoir-sur-le-widi).

Enfin, si vous voulez pousser encore plus loin vos connaissances sur le Wireless Display pour Android, vous pouvez consulter la vidéo de la conférence au Paris Android User Group sur la [présentation du WiDi pour Android](http://www.youtube.com/watch?v=gZopNFkeHH4) par [Xavier Hallade](http://ph0b.com/) ([slides ici](http://fr.slideshare.net/bemyapp/wi-di-android-applications-development))

Assez de liens, passons au développement !

<a name="intro-miraslide-2" />

MiraSlide : Une application de Présentations
--------------------------------------------

<img src="{{ site.url }}/assets/images/miraslide_logo.png" alt="MiraSlide logo" style="width: 300px; margin-left: auto; margin-right: auto"/>

{: .center }
[Code source du projet MiraSlide](https://github.com/CmoaToto/MiraSlide)

{: .center }
[Page Google Play de l'application](https://play.google.com/store/apps/details?id=fr.cmoatoto.miraslide)


Le but de cette application est de répondre à un besoin simple : les conférenciers sont soit encombrés, soit esclaves du matériel prêté, soit les deux. Ici, le Wireless Display associé à un appareil plus léger qu'un ordinateur et pouvant donc être utilisé en télécommande nous permettrait de diffuser et commander les slides depuis son téléphone/tablette, en ayant en plus dans la main les informations supplémentaires que nous apporte un ordinateur (chronomètre, notes...).

Voici donc le but très simple de MiraSlide :

1. Vous sélectionnez votre fichier de présentation, vous connectez votre appareil à un écran récepteur Wireless Display puis vous lancez la présentation.

2. Sur l'écran récepteur, la première page de votre présentation s'affiche.

3. L'écran de votre appareil vous propose alors un chronomètre, ainsi qu'une télécommande affichant le slide courant, les notes éventuelles, et des boutons précédent / suivant.

<img src="{{ site.url }}/assets/images/miraslide_presentation.png" alt="MiraSlide presentation" style="width: 600px; margin-left: auto; margin-right: auto"/>

Maintenant que nous avons une idée plus précise de l'application que nous voulons développer, nous allons nous pencher sur les APIs.

<a name="widi-dans-android-3" />

Utiliser le WiDi dans Android
-----------------------------

Pour ceux qui n'auraient pas bien lu les prérequis, le WiDi (et plus largement la notion d'écran externe) apparaît dans le framework d'Android avec l'API 17 (Android 4.2.2). Deux éléments essentiels sont alors créés :

- Le **[Display Manager](http://developer.android.com/reference/android/hardware/display/DisplayManager.html)** va être l'interface permettant à l'application de connaitre les écrans disponibles et d’interagir avec.
- Une **[Presentation](http://developer.android.com/reference/android/app/Presentation.html)** est une vue similaire à une **[Dialog](http://developer.android.com/reference/android/app/Dialog.html)** (dont elle est étendue) mais projetée sur un **[Display](http://developer.android.com/reference/android/view/Display.html)** donné. Une des notions les plus importantes à comprendre du fait que la **Presentation** est une extension d'une **Dialog** est qu'elle est forcément attachée à une **[Activity](http://developer.android.com/reference/android/app/Activity.html)**. Ainsi, si cette dernière est mise en pause (si elle n'est plus visible à l'écran, en gros), alors la **Presentation** n’apparaîtra plus sur le **Display** associé (et le mode d'écran clone par défaut s'activera). Si vous retournez ensuite à votre **Activity**, la **Presentation** reviendra s'afficher sur le **Display**.
L'implémentation est finalement très simple :

1. [Récupération du Display][31]
2. [La création et l'affichage de la Presentation][32]
3. [Ajout de listeners][33]

    [31]: #recuperation-display-31
    [32]: #creation-affichage-32
    [33]: #ajout-listeners-33

<a name="recuperation-display-31" />

### 1. La récupération du Display

Elle peut se faire de deux façons. Ou bien en utilisant le **[MediaRouter](http://developer.android.com/reference/android/media/MediaRouter.html)** introduit avec l'API 16 :

{% highlight java %}
MediaRouter mediaRouter = (MediaRouter) context.getSystemService(Context.MEDIA_ROUTER_SERVICE);
MediaRouter.RouteInfo route = mediaRouter.getSelectedRoute();

if (route != null) {
    Display presentationDisplay = route.getPresentationDisplay();

    if (presentationDisplay != null) {

        // Your code...

    }
}
{% endhighlight %}

Ou bien en utilisant le **[Display Manager](http://developer.android.com/reference/android/hardware/display/DisplayManager.html)** :

{% highlight java %}
DisplayManager displayManager = (DisplayManager) mActivity.getSystemService(Context.DISPLAY_SERVICE);

// Selecting DISPLAY_CATEGORY_PRESENTATION prevents the DisplayManager from returning inapropriate Display,
// like the own device display.
Display[] displays = displayManager.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION);

if (displays.length == 0) {

    // If there is no external display connected, we launch the Display settings. We could launch
    // the Wifi display settings with ACTION_WIFI_DISPLAY_SETTINGS but it is a hidden static value
    // because there may be not such settings (if the device does not have Wireless display but
    // have API >= 17).
    startActivity(new Intent(Settings.ACTION_DISPLAY_SETTINGS));

} else {

    // We should show a DialogBox to let the user select the display if there is more than one but
    // for this example we only choose the first one
    Display display = displays[0];

}

{% endhighlight %}

<a name="creation-affichage-32" />

### 2. La création et l'affichage de la Presentation

Ici non plus, rien de très compliqué. La **Presentation** n'a besoin que de l'**Activity** parente et du **Display** où être affiché pour être créé. Ensuite, la méthode ***show()*** affiche la **Presentation** sur le **Display**. Comme une **Dialog**, la **Presentation** comprend une méthode ***setContentView()*** grâce à laquelle vous pourrez définir la vue à afficher :

{% highlight java %}
private void showPresentation() {
    mPresentation = new MyPresentation(this, mDisplay);
    mPresentation.show();
}

private class MyPresentation extends Presentation {

    /* constructors ... */

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        View v = getLayoutInflater().inflate(R.layout.presentation, null);
        setContentView(v);
    }
}

{% endhighlight %}

<a name="ajout-listeners-33" />

### 3. Ajout de listeners

Pour rendre votre système plus fiable, vous pouvez rajouter des listeners au **DisplayManager** afin d'être prévenu lorsque des **Display** sont ajoutés ou retirés. Ceci est particulièrement pratique pour éviter à une **Presentation** de tenter de continuer à fonctionner alors que le **Display** qui lui est associé a été déconnecté :

{% highlight java %}
mDisplayManager.registerDisplayListener(new DisplayListener() {

    @Override
    public void onDisplayRemoved(int displayId) {
        // Stop presentation ...
        // Show a message to the user to reconnect the display

    }

    @Override
    public void onDisplayChanged(int displayId) {
        // Something happend. You should check if everything is ok before continuing

    }

    @Override
    public void onDisplayAdded(int displayId) {
        // If you were waiting for a display, maybe you should use it !

    }
}, null);

{% endhighlight %}

<a name="implem-miraslide-4" />

Implémentation du code dans MiraSlide
-------------------------------------

Nous n'allons bien évidemment pas revenir sur tout le code de MiraSlide, ce serait long et inutile, car finalement le code concernant le **Wireless Display** est assez court face au reste du code. Nous allons donc nous focaliser sur les points suivants :

1. [La récupération du Display][41]
2. [La sélection du Display][42]
3. [La création et l'affichage de la Presentation][43]
4. [Le contrôle de la Presentation][44]

    [41]: #recuperation-display-41
    [42]: #selection-display-42
    [43]: #creation-affichage-43
    [44]: #controle-presentation-44

Cependant, je vais rapidement expliquer la structure générale du code qui se divise en 4 éléments principaux.

- la **MainActivity** est l'unique **Activity** de l'application. Ainsi, si la **Presentation** est en cours et que l'on se déplace entre les différentes vues, la **Presentation** ne s'arrête pas.
- le **SelectionFragment** est la première vue, qui va permettre à l'utilisateur de sélectionner le fichier (PDF) contenant les slides, ainsi que le **Display** sur lequel afficher la **Presentation**. Enfin, il permet de lancer la-dite **Presentation**.
- le **ControllerFragment** est la vue s'affichant lorsque l'on lance la **Presentation**. Elle comprend un chronomètre, le slide courant ainsi que des boutons précédent et suivant.
- le **PdfViewerPresentation** est la vue **Presentation** gérant ce qui est affiché sur le **Display**. Il comprend le moteur permettant de récupérer et d'afficher les images des slides demandés par le **ControllerFragment**.

<a name="recuperation-display-41" />

### 1. La récupération du Display

- À la création du **SelectionFragment**, on récupère le **DisplayManager** et on déclare les listeners.
- Chaque fois que le **SelectionFragment** est lancé ou relancé (dans le ***onResume(...)***), on va vérifier l'état des **Display**s. Si il n'yen a aucun, on propose d'afficher les paramètres d'affichages, si il y en a un seul, on le sélectionne automatiquement, et s'il yen a plus, on propose de sélectionner le **Display** voulu.

Voici le code associé :

{% highlight java %}

public class SelectionFragment extends Fragment implements OnClickListener, DisplayListener {

    // ...

    // The display manager is the object to get information about the different displays
    private DisplayManager mDisplayManager;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // ...

        // We get the display manager to get info about the display. We also register to any change
        // about the (dis)connection of the displays.
        mDisplayManager = (DisplayManager) mActivity.getSystemService(Context.DISPLAY_SERVICE);
        mDisplayManager.registerDisplayListener(this, null);
    }

    @Override
    public void onResume() {
        super.onResume();

        // On resume, we check the state of each buttons.
        checkLaunchable(getView());
    }

    // We check the selection of the pdf file and the display, and we update the color of the buttons and we update
    // if the "Launch Projection" button should be enabled
    // @param v : the global view of the fragment
    private void checkLaunchable(View v) {
        checkDisplay((TextView) v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay));

        if (mActivity.getDislay() != null) {
            v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay).setBackgroundResource(R.drawable.button_green);
        } else {
            v.findViewById(R.id.fragment_selection_button_selectwirelessdisplay).setBackgroundResource(R.drawable.button_red);
        }

        // ...

    }

    // We check the state of the external displays, update the text of the display button, and if there is only
    // one external display, we auto select it
    private void checkDisplay(TextView displayButton) {
        Display[] displays = mDisplayManager.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION);

        if (displays.length > 1 && mActivity.getDislay() == null) {
            displayButton.setText("Select a wireless display");
        } else if (displays.length == 1) {
            mActivity.setDisplay(displays[0]);
            displayButton.setText("Display selected " + displays[0].getName());
        } else {
            mActivity.setDisplay(null);
            displayButton.setText("Connect to a wireless display");
        }
    }

    // Methods called when a display is added or removed. We change the button state if we add or remove a
    // display, and we stop the presentation if there is a display removed.

    @Override
    public void onDisplayAdded(int displayId) {
        checkLaunchable(getView());
    }

    @Override
    public void onDisplayChanged(int displayId) {
    }

    @Override
    public void onDisplayRemoved(int displayId) {
        if (mActivity.getDislay() != null && displayId == mActivity.getDislay().getDisplayId()) {
            mActivity.stopPresentation();
            checkLaunchable(getView());
        }
    }
}

{% endhighlight %}

<a name="selection-display-42" />

### 2. La sélection du Display

Comme décrit plus haut, le bouton de sélection du **Display** va évoluer en fonction des **Display**s connectés à l'appareil :

Si aucun **Display** n'est connecté, le bouton affiche ***Connect to a wireless display***. Si l'utilisateur clique dessus, le code suivant est appelé. La fenêtre des paramètres d'affichage du téléphone est alors affiché. Si l'appareil est compatible Wireless Display, un bouton **Screen mirroring**, ou **Wireless Display** ou une traduction devrait apparaitre (cf image ci dessous). En cliquant dessus, la liste des appareils visibles compatible Miracast apparait. L'utilisateur n'a plus qu'à cliquer dessus pour s'y connecter.

{% highlight java %}
if (displays.length == 0) {
    // If there is no external display connected, we launch the Display settings. We could launch the Wifi display
    // settings with ACTION_WIFI_DISPLAY_SETTINGS but it is a hidden static value because there may be not such settings
    // (if the device does not have Wireless display but have API >= 17).
    startActivity(new Intent(Settings.ACTION_DISPLAY_SETTINGS));
}

{% endhighlight %}

<img src="{{ site.url }}/assets/images/miraslide_connect.png" alt="Connecting a Wireless Display on Android" style="width: 800px; margin-left: auto; margin-right: auto"/>

- Si un seul **Display** est connecté, ou si un **Display** a déjà été sélectionné, le bouton affiche ***Display selected NOM_DU_DISPLAY***. Si l'utilisateur clique dessus, le même code que lors du cas où plusieurs **Display**s sont connectés est executé.

- Si plusieurs **Display**s sont connectés, le bouton affiche ***Select a wireless display***. Si l'utilisateur clique dessus, le code suivant est appelé. Une **Dialog** box s'ouvre, listant la liste des **Display**s disponibles. Si l'utilisateur clique sur un des **Display**s, ce dernier est alors sélectionné et on se retrouve dans le cas précédent.

{% highlight java %}
// If there is one or more external display, we show a dialog box with the list of the display.
// The user can select the display he wants, or close the dialog.
final ArrayAdapter arrayAdapter = new ArrayAdapter(mActivity, android.R.layout.select_dialog_singlechoice);
for (int i = 0; i < displays.length; i++) {
    arrayAdapter.add(displays[i].getName());
}

 AlertDialog.Builder builder = new AlertDialog.Builder(mActivity).setIcon(R.drawable.ic_launcher).setTitle("Select a display")
        .setNegativeButton("cancel", null).setAdapter(arrayAdapter, new DialogInterface.OnClickListener() {

            @Override
            public void onClick(DialogInterface dialog, int which) {
                // When the user choose a display through the dialog, we set it in the parent Activity and update
                // the state of the buttons.
                mActivity.setDisplay(displays[which]);
                checkLaunchable(getView());
            }
});
builder.show();

{% endhighlight %}

<a name="creation-affichage-43" />

### 3. La création et l'affichage de la Presentation

Lorsque le **Display** est sélectionné, ainsi qu'un fichier PDF, le bouton ***Launch Projection*** est activé. Lorsque l'utilisateur clique dessus, la **Presentation** est créée et affichée. On bascule alors l'utilisateur sur la vue **Controller**. Voici le code exécuté :

{% highlight java %}
// SelectionFragment.java

// ...


@Override
public void onClick(View v) {
    if (v.getId() == R.id.fragment_selection_button_launchprojection) {
        // On click on the "launch projection" button, we... launch the projection
        mActivity.launchPresentation();

}

{% endhighlight %}
{% highlight java %}
// MainActivity.java

// ...

// Set the presentation (created in the selection fragment). This method creates listeners to control the visibility of the controller fragment and if the
// mShowHideControllerActionBarButton should be enabled. Then it shows the presentation.
//
// @param presentation : the presentation to show
//
public void launchPresentation() {
    mPresentation = new PdfViewerPresentation(this, mDisplay, mPdfPath);

    mPresentation.setOnShowListener(new OnShowListener() {

        @Override
        public void onShow(DialogInterface dialog) {

            showHideController(true);
            enableShowHideControllerActionBarButton(true);
            mControllerFragment.notifyViewPager();
        }
    });
    mPresentation.setOnDismissListener(new OnDismissListener() {

        @Override
        public void onDismiss(DialogInterface dialog) {
            showHideController(false);
            enableShowHideControllerActionBarButton(false);
        }
    });

    mPresentation.show();
}

{% endhighlight %}

L'initialisation de la **Presentation** est très simple mais le code peut paraître un peu compliqué. Ceci est dû à la préparation du PDF et de son affichage. Ce qu'il faut retenir est que l'initialisation se fait dans le constructeur, et que la création de la vue à afficher, et l'affichage du premier slide, se fait dans le ***OnCreate(Bundle)***, et est appliqué grâce à la méthode ***setContentView(View)***. Voici le code très simplifié.

{% highlight java %}
// PdfViewerPresentation.java

// ...

// Constructor. It creates the Presentation, then load the display info and the Pdf to show
public PdfViewerPresentation(Context context, Display display, String pdfFilePath) {
    super(context, display);
    mPdfFilePath = pdfFilePath;

    // ...
}

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    createContentView();

    showPage();
}

// Create the imageView to show the pdf pages Bitmaps
private void createContentView() {
    mImageView = (ImageView) getLayoutInflater().inflate(R.layout.presentation_main, null);
    setContentView(mImageView);
}

// Show the current page on the Presentation view
private void showPage() {
    mImageView.setImageBitmap(getPage(mPage));
}

// return the Bitmap of the pdf specified page
public Bitmap getPage(int page) {

    // ...
}

{% endhighlight %}

<a name="controle-presentation-44" />

### 4. Le contrôle de la Presentation

Une fois la **Presentation** lancée, l'utilisateur peut la contrôler grâce au **ControllerFragment**. Ce dernier comprend un **ViewPager** qui affiche les slides du PDF. Lorsque l'utilisateur change de slide (soit en faisant glisser, soit en appuyant sur les boutons *Précédent* et *Suivant*), le **ControllerFragment** prévient l'**Activity** parente du nouveau slide sélectionné, et l'**Activity** fait remonter l'information à la **Presentation** afin qu'elle se mette à jour. Voici le code correspondant :

{% highlight java %}
// ControllerFragment.java

// ...

// We create the viewpager which shows the pages of the pdf file. If the user changes the slide, the listeners
// updates tell the parent Activity to change the image in the presentation.
//
// @param v : the fragment view
//
private void createViewPager(View v) {
    mSlidesPagerAdapter = new SlidesPagerAdapter(((FragmentActivity) getActivity()).getSupportFragmentManager());
    mSlidesViewPager = (ViewPager) v.findViewById(R.id.fragment_controller_pager);
    mSlidesViewPager.setAdapter(mSlidesPagerAdapter);
    mSlidesViewPager.setOnPageChangeListener(new SimpleOnPageChangeListener() {

        @Override
        public void onPageSelected(int page) {
            ((MainActivity) getActivity()).getPresentation().moveTo(page);
        }
    });
}

@Override
public void onClick(View v) {
if (v.getId() == R.id.fragment_controller_button_pageprev) {
    // On click on the prev button, we move the viewpager one slide back (which will move the presentation slide as well)
    mSlidesViewPager.setCurrentItem(mSlidesViewPager.getCurrentItem() - 1);

} else if (v.getId() == R.id.fragment_controller_button_pagenext) {
    // On click on the next button, we move the viewpager one slide next (which will move the presentation slide as well)
    mSlidesViewPager.setCurrentItem(mSlidesViewPager.getCurrentItem() + 1);
}

{% endhighlight %}
{% highlight java %}
// MainActivity.java

// ...

// return the Presentation if it has been created. A presentation needs a display and a pdf file
public PdfViewerPresentation getPresentation() {
    return mPresentation;
}

{% endhighlight %}
{% highlight java %}
// PdfViewerPresentation.java

// ...

// Move the presentation to the page 'page'
public void moveTo(int page) {
    if (page >= 0 && page < getPageCount()) {
        mPage = page;
        showPage();
    }
}

// Show the current page on the Presentation view
private void showPage() {
    mImageView.setImageBitmap(getPage(mPage));
}

// return the Bitmap of the pdf specified page
public Bitmap getPage(int page) {

    // ...
}

{% endhighlight %}

<a name="conclusion-5" />

Conclusion
----------

Développer une application exploitant les écrans externes n'est vraiment pas compliqué sous Android. Le framework est simple et fonctionne bien. Cependant, l'application développée, malgré son potentiel, est très loin de repousser le WiDi dans ses retranchements, comme pourrait le faire une application de jeu 3D, ou de streaming vidéo. Enfin, il existe une partie de l'API sur le Wireless Display qui est actuellement cachée dans le code Android et non disponible dans le SDK. Cette API permet de contrôler soi-même la découverte et la connexion aux écrans externes. L'exploitation de cette API est donc dangereuse car elle peut évoluer sans prévenir ou bien ne pas fonctionner comme espéré d'un appareil à un autre. Cependant, si cela est bien fait, l'exploitation de cette API peut simplifier grandement l'utilisation de l'application par l'utilisateur.

<a name="sources-6" />

Sources
-------

[présentation du WiDi pour Android](http://www.youtube.com/watch?v=gZopNFkeHH4) par [Xavier Hallade](http://ph0b.com/) ([slides](http://fr.slideshare.net/bemyapp/wi-di-android-applications-development))

[le site de développement Android](http://developer.android.com/index.html)

[Code source du projet MiraSlide](https://github.com/CmoaToto/MiraSlide)

[Page Google Play de l'application](https://play.google.com/store/apps/details?id=fr.cmoatoto.miraslide)
