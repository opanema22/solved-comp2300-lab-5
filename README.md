Download Link: https://assignmentchef.com/product/solved-comp2300-lab-5
<br>
Before you attend this week’s lab, make sure:

<ol>

 <li>you can read and write basic assembly code: programs with registers, instructions, labels and branching</li>

 <li>you’ve completed the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/03-maths-to-machine-code/">week 3 “pokemon” lab</a></li>

 <li>you’re familiar with the basics of functions (<a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/lectures/#chapter-3-functions">in the lectures</a>)</li>

</ol>

In this week’s lab you will:

<ol>

 <li>write <strong>functions</strong> (subroutines) to break your program into reusable components</li>

 <li>pass data in (parameters) and out (return values) of these functions</li>

 <li>keep the different parts of your code from interfering with each other (especially the registers) using the stack</li>

</ol>

<h2 id="introduction">Introduction</h2>

Imagine you have a friend who’s a teacher (or a university lecturer!) and is stressed out at the end of semester. They’ve finished marking all of their student’s assignments and exams, but the marks for these individual pieces of assessment are just scribbled around your friend’s apartment on whatever piece of paper (or wall) was closest at the time.

<img decoding="async" alt="What a mess!" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-7/scribbles-on-wall.jpg?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-7/scribbles-on-wall.jpg?w=980&amp;ssl=1" alt="What a mess!" data-recalc-dims="1">

 </noscript>

There’s only so much you can do to help your friend out, but one thing you can do is to help them add up the marks for each student’s assignments and exam to calculate their final mark for the semester.

<h3 id="functions">Functions</h3>

Functions are (usually resuable) blocks of code that have been designed to perform a specific task. Using functions allows us to break a big task (e.g. calculating the mark of a set of students) into smaller ones, which in turn can often be broken down into even smaller ones (e.g. setting and clearing a bit, potentially). How fine-grained you should break things down is a design choice that you get to make when you design your program.

The general pattern for functions look like this:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">main</span>:  <span class="hljs-comment">@ put arguments in registers</span>  <span class="hljs-comment">@ mov r0, ...</span>  <span class="hljs-keyword">bl </span>foo  <span class="hljs-comment">@ call function foo</span>  <span class="hljs-comment">@ continue here after function returns</span>  <span class="hljs-comment">@ ...</span><span class="hljs-symbol">.type</span> foo, %<span class="hljs-meta">function</span>  <span class="hljs-comment">@ optional, telling compiler foo is a function</span><span class="hljs-comment">@ args:</span><span class="hljs-comment">@   r0: ...</span><span class="hljs-comment">@ result: ...</span><span class="hljs-symbol">foo</span>:  <span class="hljs-comment">@ does something</span>  <span class="hljs-keyword">bx </span><span class="hljs-built_in">lr</span>   <span class="hljs-comment">@ return to "caller"</span><span class="hljs-symbol">.size</span> foo, .-foo      <span class="hljs-comment">@ optional, telling compiler the size of foo</span></code></pre>

You will notice that this looks very much like the label stuff we did in the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/03-maths-to-machine-code/">pokemon lab</a>—and you’d be right. Since functions are just blocks of instructions, labels are used to mark the start of functions.

The only difference between a function and the “branch to label” code you’ve written already in this course (with <code>b</code> or perhaps a conditional branch) is that with a function we want to return <em>back</em> to the <strong>caller</strong> (e.g. the <code>main</code> function) code; we branch with <code>bl</code> but we want to “come back” when we’re done with the function instructions.

That’s why <code>bl foo</code> and <code>bx lr</code> are used in the code template above instead of just <code>b foo</code>.

The <code>bl foo</code> instruction:

<ol>

 <li>records the address of the next instruction (i.e. the next value of <code>pc</code>) in the <strong>link register</strong>, and</li>

 <li>branches to the label (<code>foo</code>)</li>

</ol>

The <code>bx lr</code> instruction

<ul>

 <li>branches to the memory address stored in the <code>lr</code> register</li>

</ul>

So, together these two instructions enable branching <em>to</em> a function (<code>bl foo</code>) and branching <em>back</em> (<code>bx lr</code>) afterwards.

<p class="info-box">The <code>.type</code> and <code>.size</code> directive are optional—together they tell the compiler that the label is a function, and what the size of the function is (<code>.-foo</code> means current position minus the position of the label <code>foo</code>). They are essential for the disassembly view to work correctly for the function. They also slightly change what value the label has in instructions like <code>ldr r0, =main</code>.

<h2 id="exercise-1-a-basic-calculator">Exercise 1: a basic calculator</h2>

The assessment for your friend’s class had 3 items:

<ol>

 <li>2 assignments, marked out of 100 but worth 25% of the total mark each</li>

 <li>1 final exam, marked out of 100 but worth 50% of the total mark</li>

</ol>

As an example, here’s the marks for one student which your friend found written on a banana on the floor of his lounge room:

<table id="student-1-mark-table">

 <thead>

  <tr>

   <th>student id</th>

   <th>assignment 1</th>

   <th>assignment 2</th>

   <th>final exam</th>

  </tr>

 </thead>

 <tbody>

  <tr>

   <td>s1</td>

   <td>66</td>

   <td>73</td>

   <td>71</td>

  </tr>

 </tbody>

</table>

Your job in this exercise is to write a <code>calculate_total_mark</code> function which takes <strong>three parameters</strong> (assignment 1 score, assignment 2 score and exam score) and calculates the total mark. Be careful to take account of the “number of marks vs percentage of total mark” for each item (the maths here really isn’t tricky, but you still have to take it into account)

In this lab we’ll talk a lot about “calling functions” because that’s something you’re familiar with from higher-level programming languages. However, it’s important to remember that functions aren’t some new magical thing, they’re just a matter of using the instructions you already know in a clever way.

Plug in your discoboard, fork &amp; clone the <a class="acton-tabs-link-processed" href="https://gitlab.cecs.anu.edu.au/comp2300/2021/comp2300-2021-lab-5">lab 5 template</a> to your machine, and open the <code>src/main.S</code> file as usual. Your first job is to write a function to calculate the total mark for the student s1 provided above.

To complete this exercise, your program should:

<ol>

 <li>store the individual marks somewhere</li>

 <li>calculate the total mark</li>

 <li>put the result somewhere</li>

 <li>continue executing from where it left off before the <code>calculate_total_mark</code> function was called</li>

</ol>

As we discussed in <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/lectures/#chapter-3-functions">lectures on functions</a>, the key to packaging up a bunch of assembly instructions into a callable function is using the link register (<code>lr</code>) to remember where you branched <strong>from</strong>, and the <code>bx lr</code> instruction to jump <strong>back</strong> (or return) when you’re done.

Here’s a partial template (although you’ll have to replace the <code>??</code>s with actual assembly code for it to run:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">main</span>:  <span class="hljs-comment">@ set up the arguments</span>  <span class="hljs-keyword">mov </span><span class="hljs-built_in">r0</span>, ?? <span class="hljs-comment">@ ass1 mark</span>  <span class="hljs-keyword">mov </span><span class="hljs-built_in">r1</span>, ?? <span class="hljs-comment">@ ass2 mark</span>  <span class="hljs-keyword">mov </span><span class="hljs-built_in">r2</span>, ?? <span class="hljs-comment">@ final exam mark</span>  <span class="hljs-comment">@ call the function</span>  <span class="hljs-keyword">bl </span>calculate_total_mark  <span class="hljs-comment">@ go to the end loop</span>  <span class="hljs-keyword">b </span><span class="hljs-meta">end</span><span class="hljs-symbol">end:</span>  <span class="hljs-keyword">b </span><span class="hljs-meta">end</span><span class="hljs-symbol">calculate_total_mark:</span>  <span class="hljs-comment">@ do stuff with the arguments</span>  <span class="hljs-comment">@ ...</span>  <span class="hljs-comment">@ put the result in r0</span>  <span class="hljs-keyword">mov </span><span class="hljs-built_in">r0</span>, ??  <span class="hljs-comment">@ go back to where the function was called from</span>  <span class="hljs-keyword">bx </span>??</code></pre>

<p class="push-box">Starting with the code above, commit your a program which calculates the mark for student <code>s1</code> (see their marks in the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/05-functions/#student-1-mark-table">table above</a>), then moves into an infinite loop.

<p class="think-box">Look over your code from <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/03-maths-to-machine-code/">lab 3</a>, how would you rewrite it to use functions? <strong>it’s strongly recommended, but optional for you to give this a go</strong>… right now!

<h2 id="exercise-2-turning-marks-into-grades">Exercise 2: turning marks into grades</h2>

Your teacher friend is stoked with your solution but needs more help. They need to give a letter (<strong>A</strong> to <strong>F</strong>) grade to each student based on the following formula:

<table>

 <thead>

  <tr>

   <th>90–100</th>

   <th>80–89</th>

   <th>70–79</th>

   <th>60–69</th>

   <th>50–59</th>

   <th>0–49</th>

  </tr>

 </thead>

 <tbody>

  <tr>

   <td>A</td>

   <td>B</td>

   <td>C</td>

   <td>D</td>

   <td>E</td>

   <td>F</td>

  </tr>

 </tbody>

</table>

You tell your friend to relax—you can write another function which can do this.

In this exercise you need to write a second function called <code>grade_from_mark</code> which

<ul>

 <li><em>takes</em> a numerical mark (0–100) as input parameter</li>

 <li><em>returns</em> a value represending a letter grade (you can encode the “grade” however you like, but the hex values <code>0xA</code> to <code>0xF</code> might be a nice choice)</li>

</ul>

<p class="talk-box">There are a few ways to do this—you could generate results by doing a series of comparison tests against the different score cut-offs, but also remember that our input is a number and our output is really just a number as well. Discuss with your partner: is there a numerical transformation (a simple formula) that turns an overall mark into a grade? What are the edge cases of this formula? Are there downsides to using a “closed form solution” rather than a series of checks?

<p class="push-box">Add a <code>grade_from_mark</code> function to your program as described above. In your program, demonstrate that it returns the correct grade for the following inputs: (<code>15</code>, <code>99</code>, <code>70</code>, <code>3</code>). Commit and push your new program.

<p class="think-box">Are there any other input values which are important to check? How does your function handle “invalid” input?

<p class="extension-box">If you’re feeling adventurous, modify your program to call <code>grade_from_mark</code>, then store the result to memory in the <code>.data</code> section using the ASCII encoding.

<h2 id="exercise-3">Exercise 3: putting it together</h2>

In this exercise, you need to write a function called <code>calculate_grade</code> which combines these two steps: it takes the raw marks on the individual assessment items and returns a grade.

Write a <code>calculate_grade</code> function which calls (i.e. <code>bl</code>s) the <code>calculate_total_mark</code> function and use it to calculate the grades of the following students:

<table>

 <thead>

  <tr>

   <th>student id</th>

   <th>assignment 1</th>

   <th>assignment 2</th>

   <th>final exam</th>

  </tr>

 </thead>

 <tbody>

  <tr>

   <td>s2</td>

   <td>58</td>

   <td>51</td>

   <td>41</td>

  </tr>

  <tr>

   <td>s3</td>

   <td>68</td>

   <td>81</td>

   <td>71</td>

  </tr>

  <tr>

   <td>s4</td>

   <td>88</td>

   <td>91</td>

   <td>91</td>

  </tr>

 </tbody>

</table>

Combining these two functions is not too complicated, but remember to save your link register!

<p class="push-box">Submit a program which uses <code>calculate_grade</code> to calculate the mark of student <code>s4</code>.

<h2 id="exercise-4-recursive-functions">Exercise 4: recursive functions</h2>

Another way to implement the <code>grade_from_mark</code> function is using recursion—where a function calls <em>itself</em> over and over. Each time the function calls itself it (usually) passes itself different arguments to the time before. Still confused? <a class="acton-tabs-link-processed" href="https://www.youtube.com/watch?v=Mv9NEXX1VHc%22">Let this jolly englishman walk you through it</a>.

The basic logic for a <code>grade_from_mark_recursive</code> function is this:

<ol>

 <li>if the total mark is less than 50, the grade is a fail so the function should return (i.e. place in <code>r0</code>) the failing grade value</li>

 <li>otherwise, decrement the mark and recursively call the function passing in the new mark.</li>

</ol>

This recursive pattern will ultimately round the mark down until it hits the base case (1). After this it will then move up through the grades as the function works its way back out of the recursive calls.

Again, you need to use the stack pointer to not only keep track of your link register but also the parameters you are passing into functions so the registers don’t interfere with each other.

Your code should be something like this:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">grade_from_mark_recursive</span>:<span class="hljs-comment">@ ...</span>  <span class="hljs-keyword">bl </span>grade_from_mark_recursive  <span class="hljs-comment">@ recursive call</span><span class="hljs-comment">@ ...</span>  <span class="hljs-keyword">bx </span><span class="hljs-built_in">lr</span></code></pre>

<p class="push-box">Re-write your program from <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/05-functions/#exercise-3">Exercise 3</a> so that it calculates the grade using a recursive function.

<p class="talk-box">Discuss with your lab neighbor—what are the pros and cons between this and the original <code>grade_from_mark</code> function?

<h2 id="exercise-5-time-to-cheat">Exercise 5: time to cheat</h2>

In a new initiative, the students get to self-assess their work in the course (give themselves a final mark for the course). The only catch here is that the student’s mark is compared with the teacher’s mark. If the student mark is no more than 10 marks better than the teacher’s mark, they get the average of the two marks (i.e. theirs, and the teacher’s). If the discrepancy is more than that, they get the teacher’s mark <strong>minus</strong> the difference. This should stop any cheating—if the student’s mark is too high, they’ll actually be <em>worse</em> off than before.

Write a <code>self_assessment</code> function and incorporate it into the overall <code>calculate_grade_sa</code> function.The <code>self_assessment</code> function should return the students self-assessed grade in <code>r0</code>.

Try it with a few different versions of <code>self_assessment</code>—some which pass the “no more than 10 marks better than the teacher’s mark” criteria, and some that don’t. Does your program handle all the cases properly?

Now imagine that <em>you’re</em> the student—so you provide your own <code>self_assessment</code> function. Can you think of a way to cheat? Can you craft the assembly instructions inside the <code>self_assessment</code> function in such a way that you can get a better mark than you deserve (without touching the rest of the program)?

Use the following rough structure (still need to fill it out yourself!) of the <code>calculate_grade_sa</code> function to write the “cheating” version of <code>self_assessment</code>.

<pre><code class="language-ARM hljs"><span class="hljs-symbol">calculate_grade_sa</span>:  <span class="hljs-comment">@ <span class="hljs-doctag">TODO:</span> prep for call</span>  <span class="hljs-keyword">bl </span>calculate_total_mark  <span class="hljs-comment">@ store teacher's mark on top of stack</span>  <span class="hljs-keyword">str </span><span class="hljs-built_in">r0</span>, [<span class="hljs-built_in">sp</span>, -<span class="hljs-number">4</span>]!  <span class="hljs-comment">@ delete the teacher's mark from r0</span>  <span class="hljs-keyword">mov </span><span class="hljs-built_in">r0</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@ <span class="hljs-doctag">TODO:</span> prep for call</span>  <span class="hljs-keyword">bl </span><span class="hljs-keyword">self_assessment </span> <span class="hljs-comment">@ cheat in here</span>  <span class="hljs-keyword">ldr </span><span class="hljs-built_in">r1</span>, [<span class="hljs-built_in">sp</span>], <span class="hljs-number">4</span>  <span class="hljs-comment">@ <span class="hljs-doctag">TODO:</span> calculate final grade from: </span>  <span class="hljs-comment">@ - student grade (r0) </span>  <span class="hljs-comment">@ - teacher grade (r1)</span>  <span class="hljs-comment">@ ...</span>  <span class="hljs-keyword">bx </span><span class="hljs-built_in">lr</span><span class="hljs-symbol">self_assessment:</span>  <span class="hljs-comment">@ <span class="hljs-doctag">TODO:</span> return self assessed grade in r0</span>  <span class="hljs-comment">@ ...</span>  <span class="hljs-keyword">bx </span><span class="hljs-built_in">lr</span></code></pre>

<p class="think-box">Think about the values on the stack—can you break “outside” and mess with things outside of the <code>self_assessment</code> function? How could this allow you to cheat? <em>hint</em>: when we are using the stack pointer <code>sp</code> to store things in memory, can you figure out an offset for reading/writing values “outside” that function’s part of the stack?

<p class="do-box">There are a couple ways you can do this, can you you give yourself any arbitrary mark? How about the maximum possible mark based on the teachers final mark?

<p class="push-box">Commit &amp; push your “cheating” version of the marking program.

<p class="extension-box">This stuff is all really important for gaining a deep understanding of cybersecurity. If you are interested, you can see how the very techniques you have just learned are being applied to reverse engineering things like the <a class="acton-tabs-link-processed" href="https://www.youtube.com/watch?v=QMiubC6LdTA">Nintendo Wii U</a>!

<h2 id="exercise-6-arrays-as-arguments">Exercise 6: arrays as arguments</h2>

One of the tutors has heard about the good work you’ve been doing for your teacher friend and they have asked you to help them. Fortunately, they are more organized than the teacher and have provided you with a collection of the students results in an array–but what does an array look like in assembly? (prepare yourself for a brief tangent!)

<h3 id="sections">Sections in memory</h3>

Sections in your program are <em>directives</em> (so they start with a <code>.</code>) to the assembler that the different parts of our program should go in different parts of the discoboard’s memory space. Some parts of this address space are for instructions which the discoboard will execute, but other parts contain <em>data</em> that your program can use.

Your program can have as many sections as you like (with whatever names you like) but there are a couple of sections which the IDE &amp; toolchain will do useful things with by default:

<ul>

 <li>if you use a <code>.text</code> section in your program, then anything after that (until the next section) will appear as program code for your discoboard to execute</li>

 <li>if you use a <code>.data</code> section, then anything after that (until the next section) will be put in RAM as memory that your program can use to read/write data your program needs to do useful things</li>

</ul>

When you create a new <code>main.s</code> file, any instructions you put are put in the <code>.text</code> section until the assembler sees a new section directive.

Here’s an example:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">main</span>:  <span class="hljs-keyword">ldr </span><span class="hljs-built_in">r0</span>, <span class="hljs-symbol">=main</span>  <span class="hljs-keyword">ldr </span><span class="hljs-built_in">r1</span>, <span class="hljs-symbol">=storage</span><span class="hljs-symbol">.data</span><span class="hljs-symbol">storage</span>:  <span class="hljs-meta">.word</span> <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">0</span>, <span class="hljs-number">0</span>  <span class="hljs-meta">.asciz</span> <span class="hljs-string">"Computer Organisation &amp; Program Execution"</span></code></pre>

<p class="think-box">Looking at the discoboard’s address space map and running the program above, where do you think the <code>main</code> and <code>storage</code> parts of your program are ending up? Can you find the string “Computer Organisation &amp; Program Execution” in memory? Try and find it in the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/comp2300/labs/02-first-machine-code/#reverse-engineering">memory view</a>.

<img decoding="async" alt="Discoboard address space" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-5/address-space.jpg?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-5/address-space.jpg?w=980&amp;ssl=1" alt="Discoboard address space" data-recalc-dims="1">

 </noscript>

You can interleave the sections in your program if it makes sense:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">.text</span><span class="hljs-symbol">program</span>:  <span class="hljs-comment">@ ...</span><span class="hljs-symbol">.data</span><span class="hljs-symbol">storage</span>:  <span class="hljs-comment">@ ...</span><span class="hljs-symbol">.text</span><span class="hljs-symbol">more_program</span>:  <span class="hljs-comment">@ ...</span><span class="hljs-symbol">.data</span><span class="hljs-symbol">more_storage</span>:  <span class="hljs-comment">@ ...</span></code></pre>

When you hit build (or debug, which triggers a build) the toolchain will figure out how to put all the various bits in the right places, and you can use the labels as values in your program to make sure you’re reading and writing to the right locations.

<pre><code class="language-ARM hljs"><span class="hljs-symbol">main</span>:  <span class="hljs-keyword">ldr </span><span class="hljs-built_in">r0</span>, <span class="hljs-symbol">=results</span>  <span class="hljs-keyword">bl </span>calculate_lab_grades  <span class="hljs-keyword">nop</span>  <span class="hljs-keyword">b </span>main<span class="hljs-comment">@ ...</span><span class="hljs-comment">@ input:</span><span class="hljs-comment">@ r0: address of start of mark array with format,</span><span class="hljs-comment">@ .word size of array</span><span class="hljs-comment">@ .word a1, a2, final, 0</span><span class="hljs-comment">@ output:</span><span class="hljs-comment">@ .word a1, a2, final, grade</span><span class="hljs-comment">@ ...</span><span class="hljs-symbol">calculate_lab_grades</span>:  <span class="hljs-comment">@ ...</span>  <span class="hljs-keyword">bx </span><span class="hljs-built_in">lr</span>  <span class="hljs-comment">@ ...</span><span class="hljs-symbol">.data</span><span class="hljs-symbol">results</span>:  <span class="hljs-comment">@ Length of array: 6</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">6</span>  <span class="hljs-comment">@S1</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">50</span>, <span class="hljs-number">50</span>, <span class="hljs-number">40</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@S2</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">77</span>, <span class="hljs-number">80</span>, <span class="hljs-number">63</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@S3</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">40</span>, <span class="hljs-number">50</span>, <span class="hljs-number">60</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@S4</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">80</span>, <span class="hljs-number">82</span>, <span class="hljs-number">89</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@S5</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">80</span>, <span class="hljs-number">85</span>, <span class="hljs-number">77</span>, <span class="hljs-number">0</span>  <span class="hljs-comment">@S6</span>  <span class="hljs-meta">.word</span> <span class="hljs-number">91</span>, <span class="hljs-number">90</span>, <span class="hljs-number">95</span>, <span class="hljs-number">0</span></code></pre>

Write the <code>calculate_lab_grades</code> function to iterate over the students <code>results</code> array

<ol>

 <li>load the students results in to the registers</li>

 <li>calculate their final grade using your <code>calculate_grade</code> function (the original one, not the self assessment version)</li>

 <li>store the final grade in the empty word at the end of each entry, eg.<pre><code class="language-ARM hljs"><span class="hljs-comment">@SX</span><span class="hljs-symbol">.word</span> <span class="hljs-number">20</span>, <span class="hljs-number">40</span>, <span class="hljs-number">58</span>, <span class="hljs-number">0</span> <span class="hljs-comment">@ &lt;--- here</span></code></pre></li>

 <li>repeat for the length of the array</li>

 <li>return using <code>bx lr</code></li>

</ol>

If you’ve implemented it correctly, your memory at the results array should look like this afterwards:

<img decoding="async" alt="final grades in memory" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-7/array-in-memory.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/comp2300/assets/labs/lab-7/array-in-memory.png?w=980&amp;ssl=1" alt="final grades in memory" data-recalc-dims="1">

 </noscript>

<em>note: the final grades are stored in the 00 offset column, starting from 20000010</em>

<p class="push-box">Commit &amp; push your program to add the grades to the array.

<p class="extension-box">The values in this code are stored in memory using <code>.word</code>s which are 32 bits <em>(4 bytes)</em> in size, yet no entry needs more than a byte, can you rework your code and the array to reduce its size in memory?

5/5 - (1 vote)

If you’re interested in seeing how it’s done, you can look at your project’s linker script, located in your project folder at