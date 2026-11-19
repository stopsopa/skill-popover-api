
Before copying <FILE>, search the project for it 
(exclude dependencies directories typical to current project setup - like node_modules for node.js). 
If it is already there, use that copy and follow the comment at its top; do not copy or import it again.
If it is not there, copy it into the project, import it once, and write a short comment at the top of the copy with:
- where it is imported,
- how to use it in this particular project (framework, build setup, conventions),
- how to find the places that use it (what to search for).

