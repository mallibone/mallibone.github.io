---
layout: single
title: "Bash rename multiple files with one command"
title: Bash rename multiple files with one command
date: 2015-03-09
tags: ["General"]
slug: "bash-rename-multiple-files-with-one-command"
---

Ever had the need to correct multiple filenames?


    S0501_SomeNameS0502_SomeNameS0503_SomeNameS0504_SomeName


The following command will allow you to replace the substring and insert another string (or just replace it with an empty string) if needed.


    for filename in *.m4v; do newname=`echo $filename | sed 's/S05/S04/g'`; mv $filename $newname; done


Which will transform the sample list to:


    S0401_SomeNameS0402_SomeNameS0403_SomeNameS0404_SomeName


HTH
