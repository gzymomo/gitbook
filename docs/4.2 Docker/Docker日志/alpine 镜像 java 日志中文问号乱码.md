[TOC]

用 alpine 作为基础镜像构建了 jdk8 镜像，为线上业务的 java 微服务架构提供支持，但是有容器运行的 java 服务中打印的日志中一旦出现中文，就会出现诸如以下的 ???? 的乱码：
![](https://img2018.cnblogs.com/blog/1726962/201908/1726962-20190809104038118-1565406043.png)

# 1、使用 alpine 构建镜像时，在 dockerfile 修改其语言环境
```bash
FROM alpine:3.6

# ---not shown here---

# Install language pack
RUN apk --no-cache add ca-certificates wget && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-bin-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-i18n-2.25-r0.apk && \
    apk add glibc-bin-2.25-r0.apk glibc-i18n-2.25-r0.apk glibc-2.25-r0.apk

# Iterate through all locale and install it
# Note that locale -a is not available in alpine linux, use `/usr/glibc-compat/bin/locale -a` instead
COPY ./locale.md /locale.md
RUN cat locale.md | xargs -i /usr/glibc-compat/bin/localedef -i {} -f UTF-8 {}.UTF-8

# Set the lang, you can also specify it as as environment variable through docker-compose.yml
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

# --- not show here---
```

同级目录下创建 locale.md 文件，将以下内容拷贝入 locale.md：
```bash
aa_DJ
aa_ER
aa_ET
af_ZA
am_ET
an_ES
ar_AE
ar_BH
ar_DZ
ar_EG
ar_IN
ar_IQ
ar_JO
ar_KW
ar_LB
ar_LY
ar_MA
ar_OM
ar_QA
ar_SA
ar_SD
ar_SY
ar_TN
ar_YE
as_IN
ast_ES
ayc_PE
az_AZ
be_BY
bem_ZM
ber_DZ
ber_MA
bg_BG
bho_IN
bn_BD
bn_IN
bo_CN
bo_IN
br_FR
brx_IN
bs_BA
byn_ER
ca_AD
ca_ES
ca_FR
ca_IT
crh_UA
csb_PL
cs_CZ
cv_RU
cy_GB
da_DK
de_AT
de_BE
de_CH
de_DE
de_LU
doi_IN
dv_MV
dz_BT
el_CY
el_GR
en_AG
en_AU
en_BW
en_CA
en_DK
en_GB
en_HK
en_IE
en_IN
en_NG
en_NZ
en_PH
en_SG
en_US
en_ZA
en_ZM
en_ZW
es_AR
es_BO
es_CL
es_CO
es_CR
es_CU
es_DO
es_EC
es_ES
es_GT
es_HN
es_MX
es_NI
es_PA
es_PE
es_PR
es_PY
es_SV
es_US
es_UY
es_VE
et_EE
eu_ES
fa_IR
ff_SN
fi_FI
fil_PH
fo_FO
fr_BE
fr_CA
fr_CH
fr_FR
fr_LU
fur_IT
fy_DE
fy_NL
ga_IE
gd_GB
gez_ER
gez_ET
gl_ES
gu_IN
gv_GB
ha_NG
he_IL
hi_IN
hne_IN
hr_HR
hsb_DE
ht_HT
hu_HU
hy_AM
ia_FR
id_ID
ig_NG
ik_CA
is_IS
it_CH
it_IT
iu_CA
ja_JP
ka_GE
kk_KZ
kl_GL
km_KH
kn_IN
kok_IN
ko_KR
ks_IN
ku_TR
kw_GB
ky_KG
lb_LU
lg_UG
li_BE
lij_IT
li_NL
lo_LA
lt_LT
lv_LV
mag_IN
mai_IN
mg_MG
mhr_RU
mi_NZ
mk_MK
ml_IN
mni_IN
mn_MN
mr_IN
ms_MY
mt_MT
my_MM
nb_NO
nds_DE
nds_NL
ne_NP
nhn_MX
niu_NU
niu_NZ
nl_AW
nl_BE
nl_NL
nn_NO
nr_ZA
nso_ZA
oc_FR
om_ET
om_KE
or_IN
os_RU
pa_IN
pa_PK
pl_PL
ps_AF
pt_BR
pt_PT
ro_RO
ru_RU
ru_UA
rw_RW
sa_IN
sat_IN
sc_IT
sd_IN
se_NO
shs_CA
sid_ET
si_LK
sk_SK
sl_SI
so_DJ
so_ET
so_KE
so_SO
sq_AL
sq_MK
sr_ME
sr_RS
ss_ZA
st_ZA
sv_FI
sv_SE
sw_KE
sw_TZ
szl_PL
ta_IN
ta_LK
te_IN
tg_TJ
th_TH
ti_ER
ti_ET
tig_ER
tk_TM
tl_PH
tn_ZA
tr_CY
tr_TR
ts_ZA
tt_RU
ug_CN
uk_UA
unm_US
ur_IN
ur_PK
uz_UZ
ve_ZA
vi_VN
wa_BE
wae_CH
wal_ET
wo_SN
xh_ZA
yi_US
yo_NG
yue_HK
zh_CN
zh_HK
zh_SG
zh_TW
zu_ZA
```
这样构建出来的 alpine 镜像就是 en_US.UTF-8 的编码环境。

# 2、构建镜像
```bash
docker build -t utf8-alpine .
```

# 3、运行构建的镜像，查看编码环境
```bash
$ docker run -itd utf8-alpine sh
$ docker exec -it d830c8e49b1c sh
/opt # env
LANGUAGE=en_US.UTF-8
HOSTNAME=d830c8e49b1c
SHLVL=1
HOME=/root
TERM=xterm
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/java/jdk/bin
LANG=en_US.UTF-8
PWD=/opt
JAVA_HOME=/usr/java/jdk
/opt # /usr/glibc-compat/bin/locale -a
……
zh_CN.utf8
zh_HK.utf8
zh_SG.utf8
zh_TW.utf8
zu_ZA.utf8
```
修改完后 java 日志中的中文也已经正常显示。
