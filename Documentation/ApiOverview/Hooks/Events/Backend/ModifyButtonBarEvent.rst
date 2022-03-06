.. include:: /Includes.rst.txt
.. index:: Events; ModifyButtonBarEvent
.. _ModifyButtonBarEvent:

====================
ModifyButtonBarEvent
====================

.. versionadded:: 12.0
   This PSR-14 Eventserves as direct replacement for the now removed hook
   :php:`$GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['Backend\Template\Components\ButtonBar']['getButtonsHook']`.

This event can be used to modify the button bar in the TYPO3
:ref:`backend module docheader <backend-gui>`.

API
===

.. include:: /CodeSnippets/Manual/Backend/ModifyButtonBarEvent.rst.txt


Example
=======

Registration of the Event in your extensions' :file:`Services.yaml`:

.. code-block:: yaml
   :caption: my_extension/Configuration/Services.yaml

   MyVendor\MyPackage\Frontend\MyEventListener:
     tags:
       - name: event.listener
         identifier: 'my-package/frontend/modify-button-bar'

The corresponding event listener class:

.. code-block:: php
   :caption: my_extension/Classes/Listener/MyEventListener.php

   use TYPO3\CMS\Backend\Template\Components\ModifyButtonBarEvent;

   class MyEventListener {

       public function __invoke(ModifyButtonBarEvent $event): void
       {
           // Do your magic here
       }
   }
