# User Interface
There are two low-level ways to draw 2d graphical objects. 
- One is via creating ParaUIObject controls (like buttons, containers, editbox). These controls are managed in C++ and created in NPL scripts. 
- The other is creating owner draw ParaUIObject, and do all the drawings with Painting API in NPL scripts, such as draw rect, lines, text, etc. 

Moreover, there are a number of high-level graphical libraries written in NPL which allows you to create 2D user interface more easily. 
- IDE controls is NPL wrapper of the low-level C++ API. It provides more versatile controls than the raw ParaUIObjects.
   - One old implementation is based on ParaUIObject controls.
   - One new implementation is based on Painting API,  in `System.Window.UIElement` namespace. 
- MCML/NPL is a HTML/JS like mark up language to create user interface. 
   - One old implementation is based on ParaUIObject controls.
   - One new implementation is based on Painting API,  in `System.Window.mcml` namespace. 

## Further Reading
- [[Low-level drawing API | DrawingAPI]]
- [[UI Controls | System.Window]]
- [[mcml]]
