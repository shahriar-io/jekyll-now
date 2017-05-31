---
layout: post
title: Updating command in Eclipse RCP 3.x
---

In action sets, the UI menus enabled with the call to setEnabled(true) and disabled with setEnabled(true). Even in action sets the UI elements updated automatically. But in Eclipse 3.x we need to call setBaseEnabled(true) of AbstractHandler to enable or disable an UI menu. Even we set setBaseEnabled(true) the UI elements won't update. To update the elements we need to run the fowwing code:

```java
    public void updateUI() {
        Display.getCurrent().asyncExec(new Runnable() {
            @Override
            public void run() {
                ICommandService cmdService = (ICommandService)PlatformUI.getWorkbench().getService(ICommandService.class);
                if (cmdService != null) {
                    Collection<String> commandIDs = cmdService.getDefinedCommandIds();
                    for (String cmdID : commandIDs) {
                        if (cmdID.startsWith("the command ID")) { //$NON-NLS-1$
                            cmdService.refreshElements(cmdID, null);
                        }
                    }
                }
            }
        });
    }
```

In action sets, there is a method called selectionChanged(IAction pAction, ISelection selection). That is no more avilable in handlers. So, you need to register the previous code to the selection service. To do that put this code in the constructor of the class.

```java
        Display.getCurrent().asyncExec(new Runnable() {
            @Override
            public void run() {
                PlatformUI.getWorkbench().getActiveWorkbenchWindow().getSelectionService().addSelectionListener(new ISelectionListener() {

                    @Override
                    public void selectionChanged(IWorkbenchPart part, ISelection selection) {
                        updateUI();
                    }
                });
            }
        });
```

N.B. : Here we use Display.getCurrent().asyncExec() because the PlatformUI is not avlilable as long as the display is created. So, asyncExec() method ensures that this runnable will be called by the display thread after the display is created.
