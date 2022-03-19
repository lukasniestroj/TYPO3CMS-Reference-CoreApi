.. include:: /Includes.rst.txt

.. _backend-modules-template-without-extbase:

=======================
Create a backend module
=======================

This page covers the backend template view, using only Core functionality
without Extbase.

.. tip::

   In rare cases, where extensive data modeling is necessary, you can
   use :ref:`Extbase templating <backend-modules-template>`.
   If in most cases it makes sense to work without Extbase.

Configuration
=============

The following example configures a backend module in the menu section
:guilabel:`System` that is only accessible for admins.

The module is entered by the method :php:`handleRequest` in the class
:php:`T3docs\Examples\Controller\AdminModuleController`, which is described
below.

.. code-block:: php
   :caption: EXT:examples/Configuration/Backend/Modules.php

   use T3docs\Examples\Controller\AdminModuleController;

   return [
       //...
       'admin_examples' => [
           'parent' => 'system',
           'position' => ['top'],
           'access' => 'admin',
           'workspaces' => 'live',
           'path' => '/module/system/example',
           'labels' => 'LLL:EXT:examples/Resources/Private/Language/AdminModule/locallang_mod.xlf',
           'extensionName' => 'Examples',
           'routes' => [
              '_default' => [
                  'target' => AdminModuleController::class . '::handleRequest',
              ],
          ],
       ],
   ];

Basic controller
================

When creating a controller without Extbase an instance of :php:`ModuleTemplate`
should be used to return the rendered template. The :php:`ModuleTemplateFactory`
will take care of creating the :php:`ModuleTemplate` for you. We therefore
inject this factory via :ref:`Dependency Injection <dependency-injection>`:

.. code-block:: php

   use TYPO3\CMS\Backend\Template\ModuleTemplateFactory;
   use TYPO3\CMS\Core\Imaging\IconFactory;

   class AdminModuleController
   {
       public function __construct(
           protected readonly ModuleTemplateFactory $moduleTemplateFactory,
           protected readonly IconFactory $iconFactory,
           // ...
       ) {
       }

      public function handleRequest(ServerRequestInterface $request): ResponseInterface
      {
          $moduleTemplate = $this->moduleTemplateFactory->create($request, 't3docs/examples');
          // doSomething();
          return $moduleTemplate;
      }
   }

The controller needs to be registered in the :file:`Configuration/Services.yaml`
with the tag `backend.controller` so that dependency injection works:

.. code-block:: yaml
   :caption: EXT:examples/Configuration/Services.yaml

   services:
      _defaults:
         autowire: true
         autoconfigure: true
         public: false

      T3docs\Examples\:
         resource: '../Classes/*'
         exclude: '../Classes/Domain/Model/*'

      T3docs\Examples\Controller\AdminModuleController:
         tags: ['backend.controller']


Main entry point
================

The main entry point is defined in :confval:`routes` of the module configuration.

In this case the method :php:`handleRequest()` is defined to be the main entry
point.

.. code-block:: php
   :caption: T3docs\Examples\Controller\AdminModuleController

   public function handleRequest(ServerRequestInterface $request): ResponseInterface
   {
       $languageService = $this->getLanguageService();
       $languageService->includeLLFile('EXT:examples/Resources/Private/Language/AdminModule/locallang.xlf');

       $allowedOptions = [
           'function' => [
               'debug' => $languageService->getLL('debug'),
               'password' => $languageService->getLL('password'),
           ],
       ];

       $moduleData = $request->getAttribute('moduleData');
       if ($moduleData->cleanUp($allowedOptions)) {
           $backendUser->pushModuleData($this->moduleData->getModuleIdentifier(), $this->moduleData->toArray());
       }

       $this->menuConfig($request);
       $moduleTemplate = $this->moduleTemplateFactory->create($request, 't3docs/examples');
       $this->setUpDocHeader($moduleTemplate);

       $title = $languageService->sL('LLL:EXT:examples/Resources/Private/Language/AdminModule/locallang_mod.xlf:mlang_tabs_tab');
       switch ($moduleData->get('function')) {
           case 'debug':
               $moduleTemplate->setTitle($title, $languageService->getLL('module.menu.debug'));
               return $this->debugAction($request, $moduleTemplate);
           case 'password':
               $moduleTemplate->setTitle($title, $languageService->getLL('module.menu.password'));
               return $this->passwordAction($request, $moduleTemplate);
           default:
               $moduleTemplate->setTitle($title, $languageService->getLL('module.menu.log'));
               return $this->logAction($request, $moduleTemplate);
       }
   }

Actions
=======

Now create an example :php:`indexAction()` and assign variables to your view
as you would do for a frontend plugin.

.. code-block:: php
   :caption: T3docs\Examples\Controller\AdminModuleController

    public function indexAction(
        ServerRequestInterface $request,
        ModuleTemplate $view
    ) : ResponseInterface
    {
        $view->assign('aVariable', 'aValue');
        return $view->renderResponse('AdminModule/Index');
    }

The following example is a little more complex and demonstrates how to access
cookies and GET parameters:

.. code-block:: php
   :caption: T3docs\Examples\Controller\AdminModuleController

  protected function debugAction(
        ServerRequestInterface $request,
        ModuleTemplate $view
    ): ResponseInterface
    {
        $cmd = $request->getParsedBody()['tx_examples_admin_examples']['cmd'] ?? 'cookies';
        switch ($cmd) {
            case 'cookies':
                $this->debugCookies();
                break;
            default:
                // do something else
        }

        $view->assignMultiple(
            [
                'cookies' => $request->getCookieParams(),
                'lastcommand' => $cmd,
            ]
        );
        return $view->renderResponse('AdminModule/Debug');
    }

The DocHeader
=============

To add a DocHeader button use :php:`$this->moduleTeamplate->getDocHeaderComponent()->getButtonBar()`
and :php:`makeLinkButton()` to create the button. Finally use :php:`addButton()` to add it.

.. code-block:: php

    private function setUpDocHeader(
        ServerRequestInterface $request,
        ModuleTemplate $view
    ) {
        $buttonBar = $view->getDocHeaderComponent()->getButtonBar();
        $list = $buttonBar->makeLinkButton()
            ->setHref('<uri-builder-path>')
            ->setTitle('A Title')
            ->setShowLabelText('Link')
            ->setIcon($this->iconFactory->getIcon('actions-extension-import', Icon::SIZE_SMALL));
        $buttonBar->addButton($list, ButtonBar::BUTTON_POSITION_LEFT, 1);
    }

Template example
================

.. code-block:: html
   :caption: EXT:examples/Resources/Private/Templates/AdminModule/Debug.html

   <html data-namespace-typo3-fluid="true" xmlns:f="http://typo3.org/ns/TYPO3/CMS/Fluid/ViewHelpers">

   <f:layout name="Module" />

   <f:section name="Content">
      <h1><f:translate key="LLL:EXT:examples/Resources/Private/Language/locallang_examples.xlf:function_debug"/></h1>
      <p><f:translate key="LLL:EXT:examples/Resources/Private/Language/locallang_examples.xlf:function_debug_intro"/></p>
      <p><f:debug inline="1">{cookies}</f:debug></p>
   </f:section>
   </html>


.. note:: Some Fluid tags do not work in non-Extbase context such as
   :html:`<f:form>`.
