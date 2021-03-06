---
title: dtrace
date: 2018-12-13 09:35:24
tags:
---

### 常用命令：

```sh
$ sudo dtrace -n 'proc:::signal-send /pid/ { printf("%s -%d %d",execname,args[2],args[1]->pr_pid); ustack(); }'

$ sudo dtrace -n 'proc:::exec-success { printf("%Y %s\n",walltimestamp,curpsinfo->pr_psargs); ustack(); }'
```

<!-- more -->

### DTrace参考资料:
> Hooked on DTrace part1,2,3,4: http://blog.bignerdranch.com/category/dtrace/

### DTrace监控新启进程及其命令行参数
```
#!/usr/sbin/dtrace -C -s

#pragma D option quiet

proc:::exec-success
{
this->isx64=(curproc->p_flag & P_LP64)!=0;
#define SELECT_64_86(x64, x86) (this->isx64 ? (x64) : (x86))
#define GET_POINTER(base, offset) (user_addr_t)SELECT_64_86(*(uint64_t*)((base)+sizeof(uint64_t)*(offset)), *(uint32_t*)((base)+sizeof(uint32_t)*(offset)))

this->ptrsize=SELECT_64_86(sizeof(uint64_t),sizeof(uint32_t));
this->argc=curproc->p_argc;

// I havn't recognized whether the x64 occurs tha same problem (argv\[0\] points invalid area)
this->isClean=SELECT_64_86(1, (curproc->p_dtrace_argv==(uregs[R_SP],sizeof(uint32_t),sizeof(uint32_t))));
this->argv=(uint64_t)copyin(curproc->p_dtrace_argv,this->ptrsize*this->argc);

/* printf("%s with args:%d (%p, %p)\n",execname, this->argc, curproc->pdtraceargv, uregs\[R_SP\]); */

printf("\nargc: %d\n", this->argc);
printf("%s ", (0 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv,0)) : "");
printf("%s ", (1 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 1)) : "");
printf("%s ", (2 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 2)) : "");
printf("%s ", (3 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 3)) : "");
printf("%s ", (4 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 4)) : "");
printf("%s ", (5 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 5)) : "");
printf("%s ", (6 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 6)) : "");
printf("%s ", (7 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 7)) : "");
printf("%s ", (8 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 8)) : "");
printf("%s ", (9 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 9)) : "");
printf("%s ", (10 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 10)) : "");
printf("%s ", (11 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 11)) : "");
printf("%s ", (12 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 12)) : "");
printf("%s ", (13 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 13)) : "");
printf("%s ", (14 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 14)) : "");
printf("%s ", (15 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 15)) : "");
printf("%s ", (16 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 16)) : "");
printf("%s ", (17 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 17)) : "");
printf("%s ", (18 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 18)) : "");
printf("%s ", (19 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 19)) : "");
printf("%s ", (20 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 20)) : "");
printf("%s ", (21 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 21)) : "");
printf("%s ", (22 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 22)) : "");
printf("%s ", (23 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 23)) : "");
printf("%s ", (24 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 24)) : "");
printf("%s ", (25 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 25)) : "");
printf("%s ", (26 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 26)) : "");
printf("%s ", (27 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 27)) : "");
printf("%s ", (28 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 28)) : "");
printf("%s ", (29 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 29)) : "");
printf("%s ", (30 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 30)) : "");
printf("%s ", (31 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 31)) : "");
printf("%s ", (32 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 32)) : "");
printf("%s ", (33 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 33)) : "");
printf("%s ", (34 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 34)) : "");
printf("%s ", (35 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 35)) : "");
printf("%s ", (36 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 36)) : "");
printf("%s ", (37 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 37)) : "");
printf("%s ", (38 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 38)) : "");
printf("%s ", (39 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 39)) : "");
printf("%s ", (40 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 40)) : "");
printf("%s ", (41 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 41)) : "");
printf("%s ", (42 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 42)) : "");
printf("%s ", (43 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 43)) : "");
printf("%s ", (44 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 44)) : "");
printf("%s ", (45 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 45)) : "");
printf("%s ", (46 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 46)) : "");
printf("%s ", (47 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 47)) : "");
printf("%s ", (48 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 48)) : "");
printf("%s ", (49 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 49)) : "");
printf("%s ", (50 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 50)) : "");
printf("%s ", (51 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 51)) : "");
printf("%s ", (52 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 52)) : "");
printf("%s ", (53 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 53)) : "");
printf("%s ", (54 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 54)) : "");
printf("%s ", (55 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 55)) : "");
printf("%s ", (56 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 56)) : "");
printf("%s ", (57 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 57)) : "");
printf("%s ", (58 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 58)) : "");
printf("%s ", (59 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 59)) : "");
printf("%s ", (60 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 60)) : "");
printf("%s ", (61 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 61)) : "");
printf("%s ", (62 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 62)) : "");
printf("%s ", (63 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 63)) : "");
printf("%s ", (64 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 64)) : "");
printf("%s ", (65 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 65)) : "");
printf("%s ", (66 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 66)) : "");
printf("%s ", (67 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 67)) : "");
printf("%s ", (68 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 68)) : "");
printf("%s ", (69 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 69)) : "");
printf("%s ", (70 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 70)) : "");
printf("%s ", (71 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 71)) : "");
printf("%s ", (72 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 72)) : "");
printf("%s ", (73 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 73)) : "");
printf("%s ", (74 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 74)) : "");
printf("%s ", (75 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 75)) : "");
printf("%s ", (76 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 76)) : "");
printf("%s ", (77 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 77)) : "");
printf("%s ", (78 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 78)) : "");
printf("%s ", (79 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 79)) : "");
printf("%s ", (80 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 80)) : "");
printf("%s ", (81 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 81)) : "");
printf("%s ", (82 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 82)) : "");
printf("%s ", (83 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 83)) : "");
printf("%s ", (84 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 84)) : "");
printf("%s ", (85 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 85)) : "");
printf("%s ", (86 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 86)) : "");
printf("%s ", (87 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 87)) : "");
printf("%s ", (88 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 88)) : "");
printf("%s ", (89 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 89)) : "");
printf("%s ", (90 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 90)) : "");
printf("%s ", (91 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 91)) : "");
printf("%s ", (92 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 92)) : "");
printf("%s ", (93 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 93)) : "");
printf("%s ", (94 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 94)) : "");
printf("%s ", (95 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 95)) : "");
printf("%s ", (96 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 96)) : "");
printf("%s ", (97 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 97)) : "");
printf("%s ", (98 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 98)) : "");
printf("%s ", (99 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 99)) : "");
printf("%s ", (100 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 100)) : "");
printf("%s ", (101 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 101)) : "");
printf("%s ", (102 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 102)) : "");
printf("%s ", (103 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 103)) : "");
printf("%s ", (104 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 104)) : "");
printf("%s ", (105 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 105)) : "");
printf("%s ", (106 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 106)) : "");
printf("%s ", (107 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 107)) : "");
printf("%s ", (108 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 108)) : "");
printf("%s ", (109 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 109)) : "");
printf("%s ", (110 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 110)) : "");
printf("%s ", (111 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 111)) : "");
printf("%s ", (112 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 112)) : "");
printf("%s ", (113 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 113)) : "");
printf("%s ", (114 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 114)) : "");
printf("%s ", (115 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 115)) : "");
printf("%s ", (116 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 116)) : "");
printf("%s ", (117 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 117)) : "");
printf("%s ", (118 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 118)) : "");
printf("%s ", (119 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 119)) : "");
printf("%s ", (120 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 120)) : "");
printf("%s ", (121 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 121)) : "");
printf("%s ", (122 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 122)) : "");
printf("%s ", (123 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 123)) : "");
printf("%s ", (124 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 124)) : "");
printf("%s ", (125 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 125)) : "");
printf("%s ", (126 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 126)) : "");
printf("%s ", (127 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 127)) : "");
printf("%s ", (128 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 128)) : "");
printf("%s ", (129 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 129)) : "");
printf("%s ", (130 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 130)) : "");
printf("%s ", (131 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 131)) : "");
printf("%s ", (132 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 132)) : "");
printf("%s ", (133 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 133)) : "");
printf("%s ", (134 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 134)) : "");
printf("%s ", (135 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 135)) : "");
printf("%s ", (136 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 136)) : "");
printf("%s ", (137 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 137)) : "");
printf("%s ", (138 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 138)) : "");
printf("%s ", (139 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 139)) : "");
printf("%s ", (140 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 140)) : "");
printf("%s ", (141 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 141)) : "");
printf("%s ", (142 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 142)) : "");
printf("%s ", (143 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 143)) : "");
printf("%s ", (144 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 144)) : "");
printf("%s ", (145 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 145)) : "");
printf("%s ", (146 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 146)) : "");
printf("%s ", (147 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 147)) : "");
printf("%s ", (148 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 148)) : "");
printf("%s ", (149 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 149)) : "");
printf("%s ", (150 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 150)) : "");
printf("%s ", (151 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 151)) : "");
printf("%s ", (152 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 152)) : "");
printf("%s ", (153 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 153)) : "");
printf("%s ", (154 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 154)) : "");
printf("%s ", (155 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 155)) : "");
printf("%s ", (156 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 156)) : "");
printf("%s ", (157 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 157)) : "");
printf("%s ", (158 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 158)) : "");
printf("%s ", (159 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 159)) : "");
printf("%s ", (160 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 160)) : "");
printf("%s ", (161 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 161)) : "");
printf("%s ", (162 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 162)) : "");
printf("%s ", (163 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 163)) : "");
printf("%s ", (164 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 164)) : "");
printf("%s ", (165 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 165)) : "");
printf("%s ", (166 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 166)) : "");
printf("%s ", (167 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 167)) : "");
printf("%s ", (168 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 168)) : "");
printf("%s ", (169 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 169)) : "");
printf("%s ", (170 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 170)) : "");
printf("%s ", (171 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 171)) : "");
printf("%s ", (172 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 172)) : "");
printf("%s ", (173 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 173)) : "");
printf("%s ", (174 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 174)) : "");
printf("%s ", (175 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 175)) : "");
printf("%s ", (176 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 176)) : "");
printf("%s ", (177 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 177)) : "");
printf("%s ", (178 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 178)) : "");
printf("%s ", (179 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 179)) : "");
printf("%s ", (180 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 180)) : "");
printf("%s ", (181 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 181)) : "");
printf("%s ", (182 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 182)) : "");
printf("%s ", (183 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 183)) : "");
printf("%s ", (184 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 184)) : "");
printf("%s ", (185 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 185)) : "");
printf("%s ", (186 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 186)) : "");
printf("%s ", (187 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 187)) : "");
printf("%s ", (188 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 188)) : "");
printf("%s ", (189 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 189)) : "");
printf("%s ", (190 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 190)) : "");
printf("%s ", (191 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 191)) : "");
printf("%s ", (192 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 192)) : "");
printf("%s ", (193 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 193)) : "");
printf("%s ", (194 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 194)) : "");
printf("%s ", (195 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 195)) : "");
printf("%s ", (196 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 196)) : "");
printf("%s ", (197 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 197)) : "");
printf("%s ", (198 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 198)) : "");
printf("%s ", (199 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 199)) : "");
printf("%s ", (200 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 200)) : "");
printf("%s ", (201 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 201)) : "");
printf("%s ", (202 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 202)) : "");
printf("%s ", (203 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 203)) : "");
printf("%s ", (204 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 204)) : "");
printf("%s ", (205 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 205)) : "");
printf("%s ", (206 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 206)) : "");
printf("%s ", (207 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 207)) : "");
printf("%s ", (208 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 208)) : "");
printf("%s ", (209 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 209)) : "");
printf("%s ", (210 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 210)) : "");
printf("%s ", (211 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 211)) : "");
printf("%s ", (212 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 212)) : "");
printf("%s ", (213 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 213)) : "");
printf("%s ", (214 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 214)) : "");
printf("%s ", (215 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 215)) : "");
printf("%s ", (216 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 216)) : "");
printf("%s ", (217 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 217)) : "");
printf("%s ", (218 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 218)) : "");
printf("%s ", (219 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 219)) : "");
printf("%s ", (220 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 220)) : "");
printf("%s ", (221 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 221)) : "");
printf("%s ", (222 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 222)) : "");
printf("%s ", (223 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 223)) : "");
printf("%s ", (224 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 224)) : "");
printf("%s ", (225 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 225)) : "");
printf("%s ", (226 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 226)) : "");
printf("%s ", (227 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 227)) : "");
printf("%s ", (228 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 228)) : "");
printf("%s ", (229 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 229)) : "");
printf("%s ", (230 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 230)) : "");
printf("%s ", (231 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 231)) : "");
printf("%s ", (232 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 232)) : "");
printf("%s ", (233 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 233)) : "");
printf("%s ", (234 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 234)) : "");
printf("%s ", (235 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 235)) : "");
printf("%s ", (236 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 236)) : "");
printf("%s ", (237 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 237)) : "");
printf("%s ", (238 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 238)) : "");
printf("%s ", (239 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 239)) : "");
printf("%s ", (240 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 240)) : "");
printf("%s ", (241 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 241)) : "");
printf("%s ", (242 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 242)) : "");
printf("%s ", (243 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 243)) : "");
printf("%s ", (244 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 244)) : "");
printf("%s ", (245 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 245)) : "");
printf("%s ", (246 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 246)) : "");
printf("%s ", (247 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 247)) : "");
printf("%s ", (248 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 248)) : "");
printf("%s ", (249 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 249)) : "");
printf("%s ", (250 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 250)) : "");
printf("%s ", (251 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 251)) : "");
printf("%s ", (252 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 252)) : "");
printf("%s ", (253 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 253)) : "");
printf("%s ", (254 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 254)) : "");
printf("%s ", (255 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 255)) : "");
printf("%s ", (256 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 256)) : "");
printf("%s ", (257 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 257)) : "");
printf("%s ", (258 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 258)) : "");
printf("%s ", (259 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 259)) : "");
printf("%s ", (260 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 260)) : "");
printf("%s ", (261 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 261)) : "");
printf("%s ", (262 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 262)) : "");
printf("%s ", (263 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 263)) : "");
printf("%s ", (264 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 264)) : "");
printf("%s ", (265 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 265)) : "");
printf("%s ", (266 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 266)) : "");
printf("%s ", (267 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 267)) : "");
printf("%s ", (268 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 268)) : "");
printf("%s ", (269 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 269)) : "");
printf("%s ", (270 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 270)) : "");
printf("%s ", (271 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 271)) : "");
printf("%s ", (272 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 272)) : "");
printf("%s ", (273 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 273)) : "");
printf("%s ", (274 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 274)) : "");
printf("%s ", (275 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 275)) : "");
printf("%s ", (276 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 276)) : "");
printf("%s ", (277 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 277)) : "");
printf("%s ", (278 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 278)) : "");
printf("%s ", (279 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 279)) : "");
printf("%s ", (280 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 280)) : "");
printf("%s ", (281 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 281)) : "");
printf("%s ", (282 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 282)) : "");
printf("%s ", (283 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 283)) : "");
printf("%s ", (284 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 284)) : "");
printf("%s ", (285 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 285)) : "");
printf("%s ", (286 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 286)) : "");
printf("%s ", (287 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 287)) : "");
printf("%s ", (288 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 288)) : "");
printf("%s ", (289 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 289)) : "");
printf("%s ", (290 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 290)) : "");
printf("%s ", (291 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 291)) : "");
printf("%s ", (292 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 292)) : "");
printf("%s ", (293 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 293)) : "");
printf("%s ", (294 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 294)) : "");
printf("%s ", (295 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 295)) : "");
printf("%s ", (296 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 296)) : "");
printf("%s ", (297 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 297)) : "");
printf("%s ", (298 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 298)) : "");
printf("%s ", (299 < this->argc && this->isClean) ? copyinstr(GET_POINTER(this->argv, 299)) : "");
printf("\n");

#undef GET_POINTER
#undef SELECT_64_86
}
```
