# About #

This class is for debugging and logging Java variables to the chrome console.<br>
It is server side library for Chrome Logger extension, developed by <a href='http://craig.is/'>Craig Campbell</a><br>
You can find it (and installation instructions) here -> <a href='http://www.chromelogger.com'>http://www.chromelogger.com</a><br>
<br>
This class uses<br>
<ul><li><b>Base64</b> library - <a href='http://www.sauronsoftware.it/projects/javabase64/'>http://www.sauronsoftware.it/projects/javabase64/</a>
</li><li><b>JSON.simple</b> library - <a href='http://code.google.com/p/json-simple/'>http://code.google.com/p/json-simple/</a></li></ul>

<h1>Usage</h1>

1. <a href='https://chrome.google.com/webstore/detail/chrome-logger/noaneddfkdjfnfdakjjmocngnfkfehhd'>Download</a> and install the Chrome Logger extension from the Google Chrome Extension Gallery<br>
<br>
2. Click the extension icon to enable logging for the current tab's domain<br>
<blockquote><img src='https://googledrive.com/host/0BysQh6r0faC9NUdvZC1mSkJObVU/0.png' /></blockquote>

3. Download the ChromeLogger4J class for logging, add it to your classpath and import it.<br>Note, that class have compiled using JDK 6, so you may want to recompile it.<br>
<br>
4. Start logging in your servlet<br>
<pre><code>public void service(HttpServletRequest request, HttpServletResponse response)<br>
 throws ServletException, IOException {<br>
<br>
    ChromeLogger4J console = new ChromeLogger4J(response);<br>
    <br>
    ArrayList&lt;String&gt; strArr = new ArrayList&lt;String&gt;() {{<br>
        add("line1");<br>
        add("line2");<br>
        add("line3");<br>
    }};<br>
    <br>
    console.stack = true;<br>
    console.log(strArr);<br>
    ...<br>
</code></pre>

5. If all is working correctly then your output will look something like this<br>
<blockquote><img src='https://googledrive.com/host/0BysQh6r0faC9NUdvZC1mSkJObVU/1.png' /><br>
By default, Chrome Logger extension has «show line numbers» option switched off, but if you'll switch it on, remember to set console.stack to true, as showed above. Otherwise instead of call stack you will see «stack->false» message. By default, stack field is false to decrease size of sending data.</blockquote>

6. Also you can log variables of other types, even nested<br>
<pre><code>final HashMap&lt;String,String&gt; strMap = new HashMap&lt;String,String&gt;() {{<br>
    put("name", "Vasya");<br>
    put("surname", "Pupkin");<br>
    put("email", "vasya.pupkin@mailinator.com");<br>
}};<br>
<br>
console.log(strMap);<br>
<br>
HashMap&lt;String,Object&gt; map = new HashMap&lt;String,Object&gt;() {{<br>
    put("company", "Company Inc");<br>
    put("director", strMap);<br>
    put("since", new Date());<br>
}};<br>
<br>
console.log(map);<br>
</code></pre>
<blockquote><img src='https://googledrive.com/host/0BysQh6r0faC9NUdvZC1mSkJObVU/2.png' /></blockquote>

7. For unknown types you can use reflection (NB! It may crash response if there are too many recursive fields in your object!)<br>
<pre><code>console.reflect = true;<br>
console.reflectmethods = true;<br>
console.warn("console", console);<br>
</code></pre>
<blockquote><img src='https://googledrive.com/host/0BysQh6r0faC9NUdvZC1mSkJObVU/3.png' /><br>
Here you can see also warning type of message</blockquote>

<h1>Why?</h1>

It may look weird to debug a servlet by this way, I agree. And, in general, it is not convenient way to watch your variables.<br>
But! If in your project (like in mine) your servlet is parsing and executing other script code (for example in <a href='http://en.wikipedia.org/wiki/JavaScript'>JavaScript</a> via <a href='http://en.wikipedia.org/wiki/Rhino_(JavaScript_engine)'>Rhino</a>, or in <a href='http://en.wikipedia.org/wiki/BeanShell'>BeanShell</a>) - this debugging way become very nice :) You can define ChromeLogger4J object «console» in script's scope and use it! From my point of view it is much more comfortable than printing to stdout and tailing webserver's log file.<br>
<br>
<h1>Known issues</h1>

1. As it said <a href='http://craig.is/writing/chrome-logger/techspecs'>here</a>, maximum data size limited to 250kb for chrome. But besides chrome, servlet container usually has own header size limit also. In this library there is no check of header size, so, if you send too much data, you'll see exception «header full: java.lang.ArrayIndexOutOfBoundsException»…<br>
<br>
2. There are five lines of stacktrace printed out, instead of one single line with file name and line number. That is because if I use console.log() call from Rhino scripts, evaluted from servlet, stack looks like<br>
<pre><code>org.mozilla.javascript.MemberBox.invoke(MemberBox.java:126)<br>
org.mozilla.javascript.NativeJavaMethod.call(NativeJavaMethod.java:225)<br>
org.mozilla.javascript.optimizer.OptRuntime.call1(OptRuntime.java:32)<br>
org.mozilla.javascript.gen.C__chromelogger4j_test_rjs_25._c_script_0(C:\chromelogger4j\test.rjs:2)<br>
org.mozilla.javascript.ContextFactory.doTopCall(ContextFactory.java:394)<br>
</code></pre>
and I don't know yet how to take only single necessary file, from which log was called.