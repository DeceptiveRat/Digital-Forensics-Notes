``` data
$ strings -td -a gotham.raw > strings.txt
$ strings -td -el -a gotham.raw >> strings.txt
$ cat strings.txt | grep -F "bi0sctf"
[REMOVED]
815775385 <h2 id="Flag"><a href="#Flag" class="headerlink" title="Flag:"></a>Flag:</h2><p><code>bi0sctf&#123;H4Ha_N0w_Th4t_1s_Th3_Punchl1n3_0f_Th3_J0k3_1snt_1t?_2d9fe9&#125;</code></p>
[REMOVED]
```
- the flag decodes to: bi0sctf{H4Ha_N0w_Th4t_1s_Th3_Punchl1n3_0f_Th3_J0k3_1snt_1t?_2d9fe9}

*this is probably not how it's intended to be solved, but I can't find a writeup saying otherwise*