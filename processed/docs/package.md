# Package

## Completion Design View

Status: designing...

Entry Point Handlers
User types a character ("UT") :position, character

UT -> AWT/Swing -> kit.ExtDefaultKeyAction
  lockDocument
  detachCompletionDocumentHandler
  detachCompletionCaretHandler
  updateDocument
  switch lexsupport.codeCompletion
    cc.autoShow
    cc.refresh
    cc.hide
  attachCompletionCaretHandler
  attachCompletionDocumentHandler
  unlockDocument

User moves caret ("UM") (navigation keys or mouse click) :position

UM -> AWT/Swing -> cc.CaretListener
  cc.cancel()

User invokes code assistent ("UC") :position

UC -> AWT/Swing -> kit.AssistantAction
  cc.show

User invokes kill assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

### Entry Point Handlers

User types a character ("UT") :position, character

UT -> AWT/Swing -> kit.ExtDefaultKeyAction
  lockDocument
  detachCompletionDocumentHandler
  detachCompletionCaretHandler
  updateDocument
  switch lexsupport.codeCompletion
    cc.autoShow
    cc.refresh
    cc.hide
  attachCompletionCaretHandler
  attachCompletionDocumentHandler
  unlockDocument

User moves caret ("UM") (navigation keys or mouse click) :position

UM -> AWT/Swing -> cc.CaretListener
  cc.cancel()

User invokes code assistent ("UC") :position

UC -> AWT/Swing -> kit.AssistantAction
  cc.show

User invokes kill assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
UT -> AWT/Swing -> kit.ExtDefaultKeyAction
  lockDocument
  detachCompletionDocumentHandler
  detachCompletionCaretHandler
  updateDocument
  switch lexsupport.codeCompletion
    cc.autoShow
    cc.refresh
    cc.hide
  attachCompletionCaretHandler
  attachCompletionDocumentHandler
  unlockDocument
```

User moves caret ("UM") (navigation keys or mouse click) :position

UM -> AWT/Swing -> cc.CaretListener
  cc.cancel()

User invokes code assistent ("UC") :position

UC -> AWT/Swing -> kit.AssistantAction
  cc.show

User invokes kill assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
UM -> AWT/Swing -> cc.CaretListener
  cc.cancel()
```

User invokes code assistent ("UC") :position

UC -> AWT/Swing -> kit.AssistantAction
  cc.show

User invokes kill assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
UC -> AWT/Swing -> kit.AssistantAction
  cc.show
```

User invokes kill assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
```

User invokes select assistant action

UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
UC -> AWT/Swing -> kit.AssistantAction
  cc.hide
  cc.perform  ??? [order is pending]
```

Module modifies document ("MM") :documentEvent
  
MM -> unknown thread -> cc.DocumentListener
  cc.refresh

Completion API

+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view


+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener

```q
MM -> unknown thread -> cc.DocumentListener
  cc.refresh
```

### Completion API

```q
+ Completion:
  + autoShow(position, character) instruct completion to open sometimes
  + show(position, character) instruct completion to open in limited time
  + refresh(position, character) if open then update content
  + refresh(documentEvent) if open then update content
  + hide() hide view
```

```q
+ CompletionEnvironment implements DocumentListener, CaretListener
  disableDocumentListener
  enableDocumentListener
  disableCaretListener
  enableCaretListener
```

