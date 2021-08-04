Odoo 很大程度上是数据驱动的，因此模块定义的很大一部分是 它管理的各种记录的定义：UI（菜单和视图）， 安全（访问权限和访问规则）、报告和普通数据都是 通过记录定义。 

## 结构 

在 Odoo 中定义数据的主要方式是通过 XML 数据文件： 广义结构 XML 数据文件的内容如下： 

- 根元素中任意数量的操作元素 `odoo`

```xml
<!-- 数据文件的根元素 --> 
<odoo> 
  <operation/> 
   ...  
</odoo> 
```

数据文件顺序执行，操作只能参考结果 之前定义的操作 

## 核心业务 



### `record`

`record`适当地定义或更新数据库记录，它具有 以下属性： 

- `model` **(required) **

  要创建（或更新）的模型的名称 

- `id`

  的 [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifier)此记录 。 它是强烈建议提供一份 

  - 用于记录创建，允许后续定义修改或 参考这个记录 
  - 用于记录修改，要修改的记录 

- `context`

  创建记录时使用的上下文 

- `forcecreate`

  在更新模式下，如果记录不存在，是否应该创建记录 

  需要 [外部 id ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)，默认为 `True`. 

### `field`

每条记录可以由 `field`标签，定义要设置的值 创建记录。  一种 `record`没有 `field`将使用所有默认值 值（创建）或什么都不做（更新）。 

一种 `field`有强制性 `name`属性，要设置的字段的名称， 以及定义值本身的各种方法： 

- **Nothing**

  如果没有为该字段提供值，则隐式 `False`将被设置 在场上。  可用于清除字段，或避免使用默认值 为领域。 

- `search`

  对于 [关系字段 ](https://www.odoo.com/documentation/10.0/reference/orm.html#reference-orm-fields-relational)，应该是  一个 [域 ](https://www.odoo.com/documentation/10.0/reference/orm.html#reference-orm-domains)场上的模型。 将评估域，使用它搜索域的模型并设置 搜索结果作为字段的值。  仅在以下情况下使用第一个结果 该字段是一个 [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)

- `ref`

  如果一个 `ref`提供了属性，其值必须是有效的  [外部 id ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)，它将被查找并设置为字段的值。

  主要是为了 [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)和 [`Reference`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Reference)领域 

- `type`

  如果一个 `type`提供了属性，用于解释和转换 字段的内容。  该字段的内容可以通过 使用外部文件 `file`属性，或通过节点的主体。 

  可用类型有：

  - `xml`,  `html`提取 `field`的孩子作为一个单一的文件，评估  任何 [外部 ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)使用表单指定的 `%(external_id)s`. `%%`可用于输出实际的 *%* 符号。

  -  `file`确保字段内容是当前有效的文件路径 模型，保存对 `module,path`作为字段值 

  - `char`将字段内容直接设置为字段的值，无需 改动 

  - `base64`

     对字段内容进行base64编码，与 `file` *将 属性* 图像数据加载到附件中的 

  - `int`将字段的内容转换为整数并将其设置为字段的 价值 

  - `float`将字段的内容转换为浮点数并将其设置为字段的 价值 

  - `list`,  `tuple`应该包含任意数量的 `value`具有相同元素 属性为 `field`，每个元素解析为 生成的元组或列表，并且生成的集合被设置为 字段值 

- `eval`

  对于以前的方法不合适的情况， `eval`属性简单地评估它提供的任何 Python 表达式，并且 将结果设置为字段的值。

  评估上下文包含各种模块（ `time`,  `datetime`, `timedelta`,  `relativedelta`)，一个解决 的函数 [外部问题   标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifiers)（ `ref`) 和当前字段的模型对象，如果 适用的 （ `obj`) 

### `delete`

这 `delete`标签可以删除之前定义的任意数量的记录。  它 具有以下属性： 

- `model`（必需的） 

  应删除指定记录的模型 

- `id`

  要删除的记录 的 [外部 ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)

- `search`

  一个 [域 ](https://www.odoo.com/documentation/10.0/reference/orm.html#reference-orm-domains)来查找模型的记录  消除 

`id`和 `search`是排他的 

### `function`

这 `function`tag 使用提供的参数调用模型上的方法。 它有两个强制参数 `model`和 `name`分别指定 要调用的方法的模型和名称。 

可以使用提供参数 `eval`（应该评估为一个序列 调用方法的参数）或 `value`元素（见 `list` 值）。 

### `workflow`

这 `workflow`标签向现有工作流发送信号。  工作流程 可以通过一个指定 `ref`属性（的 [外部ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)的  现有的工作流程）或 `value`标签返回工作流的 id。 

标签还有两个必选属性 `model`（模型链接到 工作流程）和 `action`（要发送到工作流的信号的名称）。 

## 快捷方式 

由于 Odoo 的一些重要结构模型复杂且涉及，  数据文件提供了更短的替代方法来定义它们  [记录标签 ](https://www.odoo.com/documentation/10.0/reference/data.html#reference-data-record)： 

### `menuitem`

定义一个 `ir.ui.menu`记录具有许多默认值和回退： 

- 父菜单 

  - 如果一个 `parent`属性设置好了，应该是 [外部id ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id) 另一个菜单项的，用作新项的父项 
  - 如果没有 `parent`提供，试图解释 `name`属性 作为一个 `/`- 分隔菜单名称序列并在菜单中查找位置 等级制度。  在这种解释中，中间菜单是自动的 创建 
  - 否则菜单被定义为“顶级”菜单项（*not* a menu with no parent） 

- 菜单名称 

  如果没有 `name`属性是已指定的，尝试从一个链接的操作获取菜单名称 （如果有）。  

  否则使用记录的 `id`

- 团体 

  一种 `groups`属性被解释为逗号分隔的序列  [外部标识 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifiers)为 `res.groups`楷模。 如果  [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifier)以减号 ( `-`）， 群组  被 *删除* 从菜单的组 

- `action`

  如果指定，则 `action`属性应该是 [外部ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id) 菜单打开时要执行的操作 

- `id`

  菜单项的 [外部 ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)



### `template`

创建一个 [QWeb 视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-qweb)，只需要 `arch` 视图的一部分，并允许一些 *可选* 属性： 

- `id`

  视图的 [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifier)

- `name`,  `inherit_id`,  `priority`

  与上的相应字段相同 `ir.ui.view`（注意： `inherit_id` 应该是 [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifier)） 

- `primary`

  如果设置为 `True`并结合一个 `inherit_id`, 定义视图 作为初级 

- `groups`

  逗号分隔的组 列表 [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifiers)

- `page`

  如果设置为 `"True"`，模板是一个网站页面（可链接到， 可删除） 

- `optional`

  `enabled`要么 `disabled`, 是否可以禁用视图（在 网站界面）及其默认状态。  如果未设置，则视图始终为 启用。 

### `report`

创建一个 `ir.actions.report.xml`记录一些默认值。 

大多只是代理属性到相应的字段 `ir.actions.report.xml`，而且还会自动在  更多 报告的 菜单 `model`. 

## CSV 数据文件 

XML 数据文件灵活且具有自我描述性，但在 批量创建相同模型的许多简单记录。 

对于这种情况，数据文件也可以使用 [csv ](http://en.wikipedia.org/wiki/Comma-separated_values)，这通常是这种情况  [访问权限 ](https://www.odoo.com/documentation/10.0/reference/security.html#reference-security-acl)： 

- 文件名是 `*model_name*.csv`
- 第一行列出要写入的字段，带有特殊字段 `id` 用于 [外部标识符 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-identifiers)（用于创建或更新） 
- 此后的每一行都创建一个新记录 

这是定义美国各州的数据文件的第一行 `res.country.state.csv`

```
"id","country_id:id","name","code"
state_au_1,au,"Australian Capital Territory","ACT"
state_au_2,au,"New South Wales","NSW"
state_au_3,au,"Northern Territory","NT"
state_au_4,au,"Queensland","QLD"
state_au_5,au,"South Australia","SA"
state_au_6,au,"Tasmania","TAS"
state_au_7,au,"Victoria","VIC"
state_au_8,au,"Western Australia","WA"
state_us_1,us,"Alabama","AL"
state_us_2,us,"Alaska","AK"
state_us_3,us,"Arizona","AZ"
state_us_4,us,"Arkansas","AR"
state_us_5,us,"California","CA"
state_us_6,us,"Colorado","CO"
```

以更易读的格式呈现： 

| id             | country_id:id | name                             | code |
| -------------- | ------------- | -------------------------------- | ---- |
| state_au_1     | au            | Australian Capital Territory     | ACT  |
| state_au_2     | au            | New South Wales                  | NSW  |
| state_au_3     | au            | Northern Territory               | NT   |
| state_au_4     | au            | Queensland                       | QLD  |
| state_au_5     | au            | South Australia                  | SA   |
| state_au_6     | au            | Tasmania                         | TAS  |
| state_au_7     | au            | Victoria                         | VIC  |
| state_au_8     | au            | Western Australia                | WA   |
| state_us_1     | us            | Alabama                          | AL   |
| state_us_2     | us            | Alaska                           | AK   |
| state_us_3     | us            | Arizona                          | AZ   |
| state_us_4     | us            | Arkansas                         | AR   |
| state_us_5     | us            | California                       | CA   |
| state_us_6     | us            | Colorado                         | CO   |
| state_us_7     | us            | Connecticut                      | CT   |
| state_us_8     | us            | Delaware                         | DE   |
| state_us_9     | us            | District of Columbia             | DC   |
| state_us_10    | us            | Florida                          | FL   |
| state_us_11    | us            | Georgia                          | GA   |
| state_us_12    | us            | Hawaii                           | HI   |
| state_us_13    | us            | Idaho                            | ID   |
| state_us_14    | us            | Illinois                         | IL   |
| state_us_15    | us            | Indiana                          | IN   |
| state_us_16    | us            | Iowa                             | IA   |
| state_us_17    | us            | Kansas                           | KS   |
| state_us_18    | us            | Kentucky                         | KY   |
| state_us_19    | us            | Louisiana                        | LA   |
| state_us_20    | us            | Maine                            | ME   |
| state_us_21    | us            | Montana                          | MT   |
| state_us_22    | us            | Nebraska                         | NE   |
| state_us_23    | us            | Nevada                           | NV   |
| state_us_24    | us            | New Hampshire                    | NH   |
| state_us_25    | us            | New Jersey                       | NJ   |
| state_us_26    | us            | New Mexico                       | NM   |
| state_us_27    | us            | New York                         | NY   |
| state_us_28    | us            | North Carolina                   | NC   |
| state_us_29    | us            | North Dakota                     | ND   |
| state_us_30    | us            | Ohio                             | OH   |
| state_us_31    | us            | Oklahoma                         | OK   |
| state_us_32    | us            | Oregon                           | OR   |
| state_us_33    | us            | Maryland                         | MD   |
| state_us_34    | us            | Massachusetts                    | MA   |
| state_us_35    | us            | Michigan                         | MI   |
| state_us_36    | us            | Minnesota                        | MN   |
| state_us_37    | us            | Mississippi                      | MS   |
| state_us_38    | us            | Missouri                         | MO   |
| state_us_39    | us            | Pennsylvania                     | PA   |
| state_us_40    | us            | Rhode Island                     | RI   |
| state_us_41    | us            | South Carolina                   | SC   |
| state_us_42    | us            | South Dakota                     | SD   |
| state_us_43    | us            | Tennessee                        | TN   |
| state_us_44    | us            | Texas                            | TX   |
| state_us_45    | us            | Utah                             | UT   |
| state_us_46    | us            | Vermont                          | VT   |
| state_us_47    | us            | Virginia                         | VA   |
| state_us_48    | us            | Washington                       | WA   |
| state_us_49    | us            | West Virginia                    | WV   |
| state_us_50    | us            | Wisconsin                        | WI   |
| state_us_51    | us            | Wyoming                          | WY   |
| state_us_as    | us            | American Samoa                   | AS   |
| state_us_fm    | us            | Federated States of Micronesia   | FM   |
| state_us_gu    | us            | Guam                             | GU   |
| state_us_mh    | us            | Marshall Islands                 | MH   |
| state_us_mp    | us            | Northern Mariana Islands         | MP   |
| state_us_pw    | us            | Palau                            | PW   |
| state_us_pr    | us            | Puerto Rico                      | PR   |
| state_us_vi    | us            | Virgin Islands                   | VI   |
| state_us_aa    | us            | Armed Forces Americas            | AA   |
| state_us_ae    | us            | Armed Forces Europe              | AE   |
| state_us_ap    | us            | Armed Forces Pacific             | AP   |
| state_br_ac    | br            | Acre                             | AC   |
| state_br_al    | br            | Alagoas                          | AL   |
| state_br_ap    | br            | Amapá                            | AP   |
| state_br_am    | br            | Amazonas                         | AM   |
| state_br_ba    | br            | Bahia                            | BA   |
| state_br_ce    | br            | Ceará                            | CE   |
| state_br_df    | br            | Distrito Federal                 | DF   |
| state_br_es    | br            | Espírito Santo                   | ES   |
| state_br_go    | br            | Goiás                            | GO   |
| state_br_ma    | br            | Maranhão                         | MA   |
| state_br_mt    | br            | Mato Grosso                      | MT   |
| state_br_ms    | br            | Mato Grosso do Sul               | MS   |
| state_br_mg    | br            | Minas Gerais                     | MG   |
| state_br_pa    | br            | Pará                             | PA   |
| state_br_pb    | br            | Paraíba                          | PB   |
| state_br_pr    | br            | Paraná                           | PR   |
| state_br_pe    | br            | Pernambuco                       | PE   |
| state_br_pi    | br            | Piauí                            | PI   |
| state_br_rj    | br            | Rio de Janeiro                   | RJ   |
| state_br_rn    | br            | Rio Grande do Norte              | RN   |
| state_br_rs    | br            | Rio Grande do Sul                | RS   |
| state_br_ro    | br            | Rondônia                         | RO   |
| state_br_rr    | br            | Roraima                          | RR   |
| state_br_sc    | br            | Santa Catarina                   | SC   |
| state_br_sp    | br            | São Paulo                        | SP   |
| state_br_se    | br            | Sergipe                          | SE   |
| state_br_to    | br            | Tocantins                        | TO   |
| state_ru_ad    | ru            | Republic of Adygeya              | AD   |
| state_ru_al    | ru            | Altai Republic                   | AL   |
| state_ru_alt   | ru            | Altai Krai                       | ALT  |
| state_ru_amu   | ru            | Amur Oblast                      | AMU  |
| state_ru_ark   | ru            | Arkhangelsk Oblast               | ARK  |
| state_ru_ast   | ru            | Astrakhan Oblast                 | AST  |
| state_ru_ba    | ru            | Republic of Bashkortostan        | BA   |
| state_ru_bel   | ru            | Belgorod Oblast                  | BEL  |
| state_ru_bry   | ru            | Bryansk Oblast                   | BRY  |
| state_ru_bu    | ru            | Republic of Buryatia             | BU   |
| state_ru_ce    | ru            | Chechen Republic                 | CE   |
| state_ru_che   | ru            | Chelyabinsk Oblast               | CHE  |
| state_ru_chu   | ru            | Chukotka Autonomous Okrug        | CHU  |
| state_ru_cu    | ru            | Chuvash Republic                 | CU   |
| state_ru_da    | ru            | Republic of Dagestan             | DA   |
| state_ru_in    | ru            | Republic of Ingushetia           | IN   |
| state_ru_irk   | ru            | Irkutsk Oblast                   | IRK  |
| state_ru_iva   | ru            | Ivanovo Oblast                   | IVA  |
| state_ru_kam   | ru            | Kamchatka Krai                   | KAM  |
| state_ru_kb    | ru            | Kabardino-Balkarian Republic     | KB   |
| state_ru_kgd   | ru            | Kaliningrad Oblast               | KGD  |
| state_ru_kl    | ru            | Republic of Kalmykia             | KL   |
| state_ru_klu   | ru            | Kaluga Oblast                    | KLU  |
| state_ru_kc    | ru            | Karachay–Cherkess Republic       | KC   |
| state_ru_kr    | ru            | Republic of Karelia              | KR   |
| state_ru_kem   | ru            | Kemerovo Oblast                  | KEM  |
| state_ru_kha   | ru            | Khabarovsk Krai                  | KHA  |
| state_ru_kk    | ru            | Republic of Khakassia            | KK   |
| state_ru_khm   | ru            | Khanty-Mansi Autonomous Okrug    | KHM  |
| state_ru_kir   | ru            | Kirov Oblast                     | KIR  |
| state_ru_ko    | ru            | Komi Republic                    | KO   |
| state_ru_kos   | ru            | Kostroma Oblast                  | KOS  |
| state_ru_kda   | ru            | Krasnodar Krai                   | KDA  |
| state_ru_kya   | ru            | Krasnoyarsk Krai                 | KYA  |
| state_ru_kgn   | ru            | Kurgan Oblast                    | KGN  |
| state_ru_krs   | ru            | Kursk Oblast                     | KRS  |
| state_ru_len   | ru            | Leningrad Oblast                 | LEN  |
| state_ru_lip   | ru            | Lipetsk Oblast                   | LIP  |
| state_ru_mag   | ru            | Magadan Oblast                   | MAG  |
| state_ru_me    | ru            | Mari El Republic                 | ME   |
| state_ru_mo    | ru            | Republic of Mordovia             | MO   |
| state_ru_mos   | ru            | Moscow Oblast                    | MOS  |
| state_ru_mow   | ru            | Moscow                           | MOW  |
| state_ru_mur   | ru            | Murmansk Oblast                  | MUR  |
| state_ru_niz   | ru            | Nizhny Novgorod Oblast           | NIZ  |
| state_ru_ngr   | ru            | Novgorod Oblast                  | NGR  |
| state_ru_nvs   | ru            | Novosibirsk Oblast               | NVS  |
| state_ru_oms   | ru            | Omsk Oblast                      | OMS  |
| state_ru_ore   | ru            | Orenburg Oblast                  | ORE  |
| state_ru_orl   | ru            | Oryol Oblast                     | ORL  |
| state_ru_pnz   | ru            | Penza Oblast                     | PNZ  |
| state_ru_per   | ru            | Perm Krai                        | PER  |
| state_ru_pri   | ru            | Primorsky Krai                   | PRI  |
| state_ru_psk   | ru            | Pskov Oblast                     | PSK  |
| state_ru_ros   | ru            | Rostov Oblast                    | ROS  |
| state_ru_rya   | ru            | Ryazan Oblast                    | RYA  |
| state_ru_sa    | ru            | Sakha Republic (Yakutia)         | SA   |
| state_ru_sak   | ru            | Sakhalin Oblast                  | SAK  |
| state_ru_sam   | ru            | Samara Oblast                    | SAM  |
| state_ru_spe   | ru            | Saint Petersburg                 | SPE  |
| state_ru_sar   | ru            | Saratov Oblast                   | SAR  |
| state_ru_se    | ru            | Republic of North Ossetia–Alania | SE   |
| state_ru_smo   | ru            | Smolensk Oblast                  | SMO  |
| state_ru_sta   | ru            | Stavropol Krai                   | STA  |
| state_ru_sve   | ru            | Sverdlovsk Oblast                | SVE  |
| state_ru_tam   | ru            | Tambov Oblast                    | TAM  |
| state_ru_ta    | ru            | Republic of Tatarstan            | TA   |
| state_ru_tom   | ru            | Tomsk Oblast                     | TOM  |
| state_ru_tul   | ru            | Tula Oblast                      | TUL  |
| state_ru_tve   | ru            | Tver Oblast                      | TVE  |
| state_ru_tyu   | ru            | Tyumen Oblast                    | TYU  |
| state_ru_ty    | ru            | Tyva Republic                    | TY   |
| state_ru_ud    | ru            | Udmurtia                         | UD   |
| state_ru_uly   | ru            | Ulyanovsk Oblast                 | ULY  |
| state_ru_vla   | ru            | Vladimir Oblast                  | VLA  |
| state_ru_vgg   | ru            | Volgograd Oblast                 | VGG  |
| state_ru_vlg   | ru            | Vologda Oblast                   | VLG  |
| state_ru_vor   | ru            | Voronezh Oblast                  | VOR  |
| state_ru_yan   | ru            | Yamalo-Nenets Autonomous Okrug   | YAN  |
| state_ru_yar   | ru            | Yaroslavl Oblast                 | YAR  |
| state_ru_yev   | ru            | Jewish Autonomous Oblast         | YEV  |
| state_gt_ave   | gt            | Alta Verapaz                     | AVE  |
| state_gt_bve   | gt            | Baja Verapaz                     | BVE  |
| state_gt_cmt   | gt            | Chimaltenango                    | CMT  |
| state_gt_cqm   | gt            | Chiquimula                       | CQM  |
| state_gt_epr   | gt            | El Progreso                      | EPR  |
| state_gt_esc   | gt            | Escuintla                        | ESC  |
| state_gt_gua   | gt            | Guatemala                        | GUA  |
| state_gt_hue   | gt            | Huehuetenango                    | HUE  |
| state_gt_iza   | gt            | Izabal                           | IZA  |
| state_gt_jal   | gt            | Jalapa                           | JAL  |
| state_gt_jut   | gt            | Jutiapa                          | JUT  |
| state_gt_pet   | gt            | Petén                            | PET  |
| state_gt_que   | gt            | Quetzaltenango                   | QUE  |
| state_gt_qui   | gt            | Quiché                           | QUI  |
| state_gt_ret   | gt            | Retalhuleu                       | RET  |
| state_gt_sac   | gt            | Sacatepéquez                     | SAC  |
| state_gt_sma   | gt            | San Marcos                       | SMA  |
| state_gt_sro   | gt            | Santa Rosa                       | SRO  |
| state_gt_sol   | gt            | Sololá                           | SOL  |
| state_gt_suc   | gt            | Suchitepéquez                    | SUC  |
| state_gt_tot   | gt            | Totonicapán                      | TOT  |
| state_gt_zac   | gt            | Zacapa                           | ZAC  |
| state_jp_jp-23 | jp            | Aichi                            | 23   |
| state_jp_jp-05 | jp            | Akita                            | 05   |
| state_jp_jp-02 | jp            | Aomori                           | 02   |
| state_jp_jp-12 | jp            | Chiba                            | 12   |
| state_jp_jp-38 | jp            | Ehime                            | 38   |
| state_jp_jp-18 | jp            | Fukui                            | 18   |
| state_jp_jp-40 | jp            | Fukuoka                          | 40   |
| state_jp_jp-07 | jp            | Fukushima                        | 07   |
| state_jp_jp-21 | jp            | Gifu                             | 21   |
| state_jp_jp-10 | jp            | Gunma                            | 10   |
| state_jp_jp-34 | jp            | Hiroshima                        | 34   |
| state_jp_jp-01 | jp            | Hokkaidō                         | 01   |
| state_jp_jp-28 | jp            | Hyōgo                            | 28   |
| state_jp_jp-08 | jp            | Ibaraki                          | 08   |
| state_jp_jp-17 | jp            | Ishikawa                         | 17   |
| state_jp_jp-03 | jp            | Iwate                            | 03   |
| state_jp_jp-37 | jp            | Kagawa                           | 37   |
| state_jp_jp-46 | jp            | Kagoshima                        | 46   |
| state_jp_jp-14 | jp            | Kanagawa                         | 14   |
| state_jp_jp-39 | jp            | Kōchi                            | 39   |
| state_jp_jp-43 | jp            | Kumamoto                         | 43   |
| state_jp_jp-26 | jp            | Kyōto                            | 26   |
| state_jp_jp-24 | jp            | Mie                              | 24   |
| state_jp_jp-04 | jp            | Miyagi                           | 04   |
| state_jp_jp-45 | jp            | Miyazaki                         | 45   |
| state_jp_jp-20 | jp            | Nagano                           | 20   |
| state_jp_jp-42 | jp            | Nagasaki                         | 42   |
| state_jp_jp-29 | jp            | Nara                             | 29   |
| state_jp_jp-15 | jp            | Niigata                          | 15   |
| state_jp_jp-44 | jp            | Ōita                             | 44   |
| state_jp_jp-33 | jp            | Okayama                          | 33   |
| state_jp_jp-47 | jp            | Okinawa                          | 47   |
| state_jp_jp-27 | jp            | Ōsaka                            | 27   |
| state_jp_jp-41 | jp            | Saga                             | 41   |
| state_jp_jp-11 | jp            | Saitama                          | 11   |
| state_jp_jp-25 | jp            | Shiga                            | 25   |
| state_jp_jp-32 | jp            | Shimane                          | 32   |
| state_jp_jp-22 | jp            | Shizuoka                         | 22   |
| state_jp_jp-09 | jp            | Tochigi                          | 09   |
| state_jp_jp-36 | jp            | Tokushima                        | 36   |
| state_jp_jp-31 | jp            | Tottori                          | 31   |
| state_jp_jp-16 | jp            | Toyama                           | 16   |
| state_jp_jp-13 | jp            | Tōkyō                            | 13   |
| state_jp_jp-30 | jp            | Wakayama                         | 30   |
| state_jp_jp-06 | jp            | Yamagata                         | 06   |
| state_jp_jp-35 | jp            | Yamaguchi                        | 35   |
| state_jp_jp-19 | jp            | Yamanashi                        | 19   |
| state_pt_pt-01 | pt            | Aveiro                           | 01   |
| state_pt_pt-02 | pt            | Beja                             | 02   |
| state_pt_pt-03 | pt            | Braga                            | 03   |
| state_pt_pt-04 | pt            | Bragança                         | 04   |
| state_pt_pt-05 | pt            | Castelo Branco                   | 05   |
| state_pt_pt-06 | pt            | Coimbra                          | 06   |
| state_pt_pt-07 | pt            | Évora                            | 07   |
| state_pt_pt-08 | pt            | Faro                             | 08   |
| state_pt_pt-09 | pt            | Guarda                           | 09   |
| state_pt_pt-10 | pt            | Leiria                           | 10   |
| state_pt_pt-11 | pt            | Lisboa                           | 11   |
| state_pt_pt-12 | pt            | Portalegre                       | 12   |
| state_pt_pt-13 | pt            | Porto                            | 13   |
| state_pt_pt-14 | pt            | Santarém                         | 14   |
| state_pt_pt-15 | pt            | Setúbal                          | 15   |
| state_pt_pt-16 | pt            | Viana do Castelo                 | 16   |
| state_pt_pt-17 | pt            | Vila Real                        | 17   |
| state_pt_pt-18 | pt            | Viseu                            | 18   |
| state_pt_pt-20 | pt            | Açores                           | 20   |
| state_pt_pt-30 | pt            | Madeira                          | 30   |
| state_eg_dk    | eg            | Dakahlia                         | DK   |
| state_eg_ba    | eg            | Red Sea                          | BA   |
| state_eg_bh    | eg            | Beheira                          | BH   |
| state_eg_fym   | eg            | Faiyum                           | FYM  |
| state_eg_gh    | eg            | Gharbia                          | GH   |
| state_eg_alx   | eg            | Alexandria                       | ALX  |
| state_eg_is    | eg            | Ismailia                         | IS   |
| state_eg_gz    | eg            | Giza                             | GZ   |
| state_eg_mnf   | eg            | Monufia                          | MNF  |
| state_eg_mn    | eg            | Minya                            | MN   |
| state_eg_c     | eg            | Cairo                            | C    |
| state_eg_kb    | eg            | Qalyubia                         | KB   |
| state_eg_lx    | eg            | Luxor                            | LX   |
| state_eg_wad   | eg            | New Valley                       | WAD  |
| state_eg_shr   | eg            | Al Sharqia                       | SHR  |
| state_eg_su    | eg            | 6th of October                   | SU   |
| state_eg_suz   | eg            | Suez                             | SUZ  |
| state_eg_asn   | eg            | Aswan                            | ASN  |
| state_eg_ast   | eg            | Asyut                            | AST  |
| state_eg_bns   | eg            | Beni Suef                        | BNS  |
| state_eg_pts   | eg            | Port Said                        | PTS  |
| state_eg_dt    | eg            | Damietta                         | DT   |
| state_eg_hu    | eg            | Helwan                           | HU   |
| state_eg_js    | eg            | South Sinai                      | JS   |
| state_eg_kfs   | eg            | Kafr el-Sheikh                   | KFS  |
| state_eg_mt    | eg            | Matrouh                          | MT   |
| state_eg_kn    | eg            | Qena                             | KN   |
| state_eg_sin   | eg            | North Sinai                      | SIN  |
| state_eg_shg   | eg            | Sohag                            | SHG  |
| state_za_ec    | za            | Eastern Cape                     | EC   |
| state_za_fs    | za            | Free State                       | FS   |
| state_za_gt    | za            | Gauteng                          | GT   |
| state_za_nl    | za            | KwaZulu-Natal                    | NL   |
| state_za_lp    | za            | Limpopo                          | LP   |
| state_za_mp    | za            | Mpumalanga                       | MP   |
| state_za_nc    | za            | Northern Cape                    | NC   |
| state_za_nw    | za            | North West                       | NW   |
| state_za_wc    | za            | Western Cape                     | WC   |
| state_it_ag    | it            | Agrigento                        | AG   |
| state_it_al    | it            | Alessandria                      | AL   |
| state_it_an    | it            | Ancona                           | AN   |
| state_it_ao    | it            | Aosta                            | AO   |
| state_it_ar    | it            | Arezzo                           | AR   |
| state_it_ap    | it            | Ascoli Piceno                    | AP   |
| state_it_at    | it            | Asti                             | AT   |
| state_it_av    | it            | Avellino                         | AV   |
| state_it_ba    | it            | Bari                             | BA   |
| state_it_bt    | it            | Barletta-Andria-Trani            | BT   |
| state_it_bl    | it            | Belluno                          | BL   |
| state_it_bn    | it            | Benevento                        | BN   |
| state_it_bg    | it            | Bergamo                          | BG   |
| state_it_bi    | it            | Biella                           | BI   |
| state_it_bo    | it            | Bologna                          | BO   |
| state_it_bz    | it            | Bolzano                          | BZ   |
| state_it_bs    | it            | Brescia                          | BS   |
| state_it_br    | it            | Brindisi                         | BR   |
| state_it_ca    | it            | Cagliari                         | CA   |
| state_it_cl    | it            | Caltanissetta                    | CL   |
| state_it_cb    | it            | Campobasso                       | CB   |
| state_it_ci    | it            | Carbonia-Iglesias                | CI   |
| state_it_ce    | it            | Caserta                          | CE   |
| state_it_ct    | it            | Catania                          | CT   |
| state_it_cz    | it            | Catanzaro                        | CZ   |
| state_it_ch    | it            | Chieti                           | CH   |
| state_it_co    | it            | Como                             | CO   |
| state_it_cs    | it            | Cosenza                          | CS   |
| state_it_cr    | it            | Cremona                          | CR   |
| state_it_kr    | it            | Crotone                          | KR   |
| state_it_cn    | it            | Cuneo                            | CN   |
| state_it_en    | it            | Enna                             | EN   |
| state_it_fm    | it            | Fermo                            | FM   |
| state_it_fe    | it            | Ferrara                          | FE   |
| state_it_fi    | it            | Firenze                          | FI   |
| state_it_fg    | it            | Foggia                           | FG   |
| state_it_fc    | it            | Forlì-Cesena                     | FC   |
| state_it_fr    | it            | Frosinone                        | FR   |
| state_it_ge    | it            | Genova                           | GE   |
| state_it_go    | it            | Gorizia                          | GO   |
| state_it_gr    | it            | Grosseto                         | GR   |
| state_it_im    | it            | Imperia                          | IM   |
| state_it_is    | it            | Isernia                          | IS   |
| state_it_sp    | it            | La Spezia                        | SP   |
| state_it_aq    | it            | L'Aquila                         | AQ   |
| state_it_lt    | it            | Latina                           | LT   |
| state_it_le    | it            | Lecce                            | LE   |
| state_it_lc    | it            | Lecco                            | LC   |
| state_it_li    | it            | Livorno                          | LI   |
| state_it_lo    | it            | Lodi                             | LO   |
| state_it_lu    | it            | Lucca                            | LU   |
| state_it_mc    | it            | Macerata                         | MC   |
| state_it_mn    | it            | Mantova                          | MN   |
| state_it_ms    | it            | Massa-Carrara                    | MS   |
| state_it_mt    | it            | Matera                           | MT   |
| state_it_vs    | it            | Medio Campidano                  | VS   |
| state_it_me    | it            | Messina                          | ME   |
| state_it_mi    | it            | Milano                           | MI   |
| state_it_mo    | it            | Modena                           | MO   |
| state_it_mb    | it            | Monza e Brianza                  | MB   |
| state_it_na    | it            | Napoli                           | NA   |
| state_it_no    | it            | Novara                           | NO   |
| state_it_nu    | it            | Nuoro                            | NU   |
| state_it_og    | it            | Ogliastra                        | OG   |
| state_it_ot    | it            | Olbia-Tempio                     | OT   |
| state_it_or    | it            | Oristano                         | OR   |
| state_it_pd    | it            | Padova                           | PD   |
| state_it_pa    | it            | Palermo                          | PA   |
| state_it_pr    | it            | Parma                            | PR   |
| state_it_pv    | it            | Pavia                            | PV   |
| state_it_pg    | it            | Perugia                          | PG   |
| state_it_pu    | it            | Pesaro e Urbino                  | PU   |
| state_it_pe    | it            | Pescara                          | PE   |
| state_it_pc    | it            | Piacenza                         | PC   |
| state_it_pi    | it            | Pisa                             | PI   |
| state_it_pt    | it            | Pistoia                          | PT   |
| state_it_pn    | it            | Pordenone                        | PN   |
| state_it_pz    | it            | Potenza                          | PZ   |
| state_it_po    | it            | Prato                            | PO   |
| state_it_rg    | it            | Ragusa                           | RG   |
| state_it_ra    | it            | Ravenna                          | RA   |
| state_it_rc    | it            | Reggio Calabria                  | RC   |
| state_it_re    | it            | Reggio Emilia                    | RE   |
| state_it_ri    | it            | Rieti                            | RI   |
| state_it_rn    | it            | Rimini                           | RN   |
| state_it_rm    | it            | Roma                             | RM   |
| state_it_ro    | it            | Rovigo                           | RO   |
| state_it_sa    | it            | Salerno                          | SA   |
| state_it_ss    | it            | Sassari                          | SS   |
| state_it_sv    | it            | Savona                           | SV   |
| state_it_si    | it            | Siena                            | SI   |
| state_it_sr    | it            | Siracusa                         | SR   |
| state_it_so    | it            | Sondrio                          | SO   |
| state_it_ta    | it            | Taranto                          | TA   |
| state_it_te    | it            | Teramo                           | TE   |
| state_it_tr    | it            | Terni                            | TR   |
| state_it_to    | it            | Torino                           | TO   |
| state_it_tp    | it            | Trapani                          | TP   |
| state_it_tn    | it            | Trento                           | TN   |
| state_it_tv    | it            | Treviso                          | TV   |
| state_it_ts    | it            | Trieste                          | TS   |
| state_it_ud    | it            | Udine                            | UD   |
| state_it_va    | it            | Varese                           | VA   |
| state_it_ve    | it            | Venezia                          | VE   |
| state_it_vb    | it            | Verbano-Cusio-Ossola             | VB   |
| state_it_vc    | it            | Vercelli                         | VC   |
| state_it_vr    | it            | Verona                           | VR   |
| state_it_vv    | it            | Vibo Valentia                    | VV   |
| state_it_vi    | it            | Vicenza                          | VI   |
| state_it_vt    | it            | Viterbo                          | VT   |
| state_es_c     | es            | A Coruña (La Coruña)             | C    |
| state_es_vi    | es            | Araba/Álava                      | VI   |
| state_es_ab    | es            | Albacete                         | AB   |
| state_es_a     | es            | Alacant (Alicante)               | A    |
| state_es_al    | es            | Almería                          | AL   |
| state_es_o     | es            | Asturias                         | O    |
| state_es_av    | es            | Ávila                            | AV   |
| state_es_ba    | es            | Badajoz                          | BA   |
| state_es_pm    | es            | Illes Balears (Islas Baleares)   | PM   |
| state_es_b     | es            | Barcelona                        | B    |
| state_es_bu    | es            | Burgos                           | BU   |
| state_es_cc    | es            | Cáceres                          | CC   |
| state_es_ca    | es            | Cádiz                            | CA   |
| state_es_s     | es            | Cantabria                        | S    |
| state_es_cs    | es            | Castelló (Castellón)             | CS   |
| state_es_ce    | es            | Ceuta                            | CE   |
| state_es_cr    | es            | Ciudad Real                      | CR   |
| state_es_co    | es            | Córdoba                          | CO   |
| state_es_cu    | es            | Cuenca                           | CU   |
| state_es_gi    | es            | Girona (Gerona)                  | GI   |
| state_es_gr    | es            | Granada                          | GR   |
| state_es_gu    | es            | Guadalajara                      | GU   |
| state_es_ss    | es            | Gipuzkoa (Guipúzcoa)             | SS   |
| state_es_h     | es            | Huelva                           | H    |
| state_es_hu    | es            | Huesca                           | HU   |
| state_es_j     | es            | Jaén                             | J    |
| state_es_lo    | es            | La Rioja                         | LO   |
| state_es_gc    | es            | Las Palmas                       | GC   |
| state_es_le    | es            | León                             | LE   |
| state_es_l     | es            | Lleida (Lérida)                  | L    |
| state_es_lu    | es            | Lugo                             | LU   |
| state_es_m     | es            | Madrid                           | M    |
| state_es_ma    | es            | Málaga                           | MA   |
| state_es_ml    | es            | Melilla                          | ME   |
| state_es_mu    | es            | Murcia                           | MU   |
| state_es_na    | es            | Nafarroa (Navarra)               | NA   |
| state_es_or    | es            | Ourense (Orense)                 | OR   |
| state_es_p     | es            | Palencia                         | P    |
| state_es_po    | es            | Pontevedra                       | PO   |
| state_es_sa    | es            | Salamanca                        | SA   |
| state_es_tf    | es            | Santa Cruz de Tenerife           | TF   |
| state_es_sg    | es            | Segovia                          | SG   |
| state_es_se    | es            | Sevilla                          | SE   |
| state_es_so    | es            | Soria                            | SO   |
| state_es_t     | es            | Tarragona                        | T    |
| state_es_te    | es            | Teruel                           | TE   |
| state_es_to    | es            | Toledo                           | TO   |
| state_es_v     | es            | València (Valencia)              | V    |
| state_es_va    | es            | Valladolid                       | VA   |
| state_es_bi    | es            | Bizkaia (Vizcaya)                | BI   |
| state_es_za    | es            | Zamora                           | ZA   |
| state_es_z     | es            | Zaragoza                         | Z    |
| state_my_jhr   | my            | Johor                            | JHR  |
| state_my_kdh   | my            | Kedah                            | KDH  |
| state_my_ktn   | my            | Kelantan                         | KTN  |
| state_my_kul   | my            | Kuala Lumpur                     | KUL  |
| state_my_lbn   | my            | Labuan                           | LBN  |
| state_my_mlk   | my            | Melaka                           | MLK  |
| state_my_nsn   | my            | Negeri Sembilan                  | NSN  |
| state_my_phg   | my            | Pahang                           | PHG  |
| state_my_prk   | my            | Perak                            | PRK  |
| state_my_pls   | my            | Perlis                           | PLS  |
| state_my_png   | my            | Pulau Pinang                     | PNG  |
| state_my_pjy   | my            | Putrajaya                        | PJY  |
| state_my_sbh   | my            | Sabah                            | SBH  |
| state_my_swk   | my            | Sarawak                          | SWK  |
| state_my_sgr   | my            | Selangor                         | SGR  |
| state_my_trg   | my            | Terengganu                       | TRG  |
| state_mx_ags   | mx            | Aguascalientes                   | AGU  |
| state_mx_bc    | mx            | Baja California                  | BCN  |
| state_mx_bcs   | mx            | Baja California Sur              | BCS  |
| state_mx_chih  | mx            | Chihuahua                        | CHH  |
| state_mx_col   | mx            | Colima                           | COL  |
| state_mx_camp  | mx            | Campeche                         | CAM  |
| state_mx_coah  | mx            | Coahuila                         | COA  |
| state_mx_chis  | mx            | Chiapas                          | CHP  |
| state_mx_df    | mx            | Ciudad de México                 | DIF  |
| state_mx_dgo   | mx            | Durango                          | DUR  |
| state_mx_gro   | mx            | Guerrero                         | GRO  |
| state_mx_gto   | mx            | Guanajuato                       | GUA  |
| state_mx_hgo   | mx            | Hidalgo                          | HID  |
| state_mx_jal   | mx            | Jalisco                          | JAL  |
| state_mx_mich  | mx            | Michoacán                        | MIC  |
| state_mx_mor   | mx            | Morelos                          | MOR  |
| state_mx_mex   | mx            | México                           | MEX  |
| state_mx_nay   | mx            | Nayarit                          | NAY  |
| state_mx_nl    | mx            | Nuevo León                       | NLE  |
| state_mx_oax   | mx            | Oaxaca                           | OAX  |
| state_mx_pue   | mx            | Puebla                           | PUE  |
| state_mx_q roo | mx            | Quintana Roo                     | ROO  |
| state_mx_qro   | mx            | Querétaro                        | QUE  |
| state_mx_sin   | mx            | Sinaloa                          | SIN  |
| state_mx_slp   | mx            | San Luis Potosí                  | SLP  |
| state_mx_son   | mx            | Sonora                           | SON  |
| state_mx_tab   | mx            | Tabasco                          | TAB  |
| state_mx_tlax  | mx            | Tlaxcala                         | TLA  |
| state_mx_tamps | mx            | Tamaulipas                       | TAM  |
| state_mx_ver   | mx            | Veracruz                         | VER  |
| state_mx_yuc   | mx            | Yucatán                          | YUC  |
| state_mx_zac   | mx            | Zacatecas                        | ZAC  |
| state_nz_auk   | nz            | Auckland                         | AUK  |
| state_nz_bop   | nz            | Bay of Plenty                    | BOP  |
| state_nz_can   | nz            | Canterbury                       | CAN  |
| state_nz_gis   | nz            | Gisborne                         | GIS  |
| state_nz_hkb   | nz            | Hawke's Bay                      | HKB  |
| state_nz_mwt   | nz            | Manawatu-Wanganui                | MWT  |
| state_nz_mbh   | nz            | Marlborough                      | MBH  |
| state_nz_nsn   | nz            | Nelson                           | NSN  |
| state_nz_ntl   | nz            | Northland                        | NTL  |
| state_nz_ota   | nz            | Otago                            | OTA  |
| state_nz_stl   | nz            | Southland                        | STL  |
| state_nz_tki   | nz            | Taranaki                         | TKI  |
| state_nz_tas   | nz            | Tasman                           | TAS  |
| state_nz_wko   | nz            | Waikato                          | WKO  |
| state_nz_wgn   | nz            | Wellington                       | WGN  |
| state_nz_wtc   | nz            | West Coast                       | WTC  |
| state_ca_ab    | ca            | Alberta                          | AB   |
| state_ca_bc    | ca            | British Columbia                 | BC   |
| state_ca_mb    | ca            | Manitoba                         | MB   |
| state_ca_nb    | ca            | New Brunswick                    | NB   |
| state_ca_nl    | ca            | Newfoundland and Labrador        | NL   |
| state_ca_nt    | ca            | Northwest Territories            | NT   |
| state_ca_ns    | ca            | Nova Scotia                      | NS   |
| state_ca_nu    | ca            | Nunavut                          | NU   |
| state_ca_on    | ca            | Ontario                          | ON   |
| state_ca_pe    | ca            | Prince Edward Island             | PE   |
| state_ca_qc    | ca            | Quebec                           | QC   |
| state_ca_sk    | ca            | Saskatchewan                     | SK   |
| state_ca_yt    | ca            | Yukon                            | YT   |
| state_ae_az    | ae            | Abu Dhabi                        | AZ   |
| state_ae_aj    | ae            | Ajman                            | AJ   |
| state_ae_du    | ae            | Dubai                            | DU   |
| state_ae_fu    | ae            | Fujairah                         | FU   |
| state_ae_rk    | ae            | Ras al-Khaimah                   | RK   |
| state_ae_sh    | ae            | Sharjah                          | SH   |
| state_ae_uq    | ae            | Umm al-Quwain                    | UQ   |
| state_ar_c     | ar            | Buenos Aires City                | C    |
| state_ar_b     | ar            | Buenos Aires                     | B    |
| state_ar_k     | ar            | Catamarca                        | K    |
| state_ar_h     | ar            | Chaco                            | H    |
| state_ar_u     | ar            | Chobut                           | U    |
| state_ar_x     | ar            | Córdoba                          | X    |
| state_ar_w     | ar            | Corrientes                       | W    |
| state_ar_e     | ar            | Ente Ríos                        | E    |
| state_ar_p     | ar            | Formosa                          | P    |
| state_ar_y     | ar            | Jujuy                            | Y    |
| state_ar_l     | ar            | La Pampa                         | L    |
| state_ar_f     | ar            | La Rioja                         | F    |
| state_ar_m     | ar            | Mendoza                          | M    |
| state_ar_n     | ar            | Misiones                         | N    |
| state_ar_q     | ar            | Neuquén                          | Q    |
| state_ar_r     | ar            | Río Negro                        | R    |
| state_ar_a     | ar            | Salta                            | A    |
| state_ar_j     | ar            | San Juan                         | J    |
| state_ar_d     | ar            | San Luis                         | D    |
| state_ar_z     | ar            | Santa Cruz                       | Z    |
| state_ar_s     | ar            | Santa Fe                         | S    |
| state_ar_g     | ar            | Santiago Del Estero              | G    |
| state_ar_v     | ar            | Tierra del Fuego                 | V    |
| state_ar_t     | ar            | Tucumán                          | T    |
| state_in_an    | in            | Andaman and Nicobar              | AN   |
| state_in_ap    | in            | Andhra Pradesh                   | AP   |
| state_in_ar    | in            | Arunachal Pradesh                | AR   |
| state_in_as    | in            | Assam                            | AS   |
| state_in_br    | in            | Bihar                            | BR   |
| state_in_ch    | in            | Chandigarh                       | CH   |
| state_in_cg    | in            | Chattisgarh                      | CG   |
| state_in_dn    | in            | Dadra and Nagar Haveli           | DN   |
| state_in_dd    | in            | Daman and Diu                    | DD   |
| state_in_dl    | in            | Delhi                            | DL   |
| state_in_ga    | in            | Goa                              | GA   |
| state_in_gj    | in            | Gujarat                          | GJ   |
| state_in_hr    | in            | Haryana                          | HR   |
| state_in_hp    | in            | Himachal Pradesh                 | HP   |
| state_in_jk    | in            | Jammu and Kashmir                | JK   |
| state_in_jh    | in            | Jharkhand                        | JH   |
| state_in_ka    | in            | Karnataka                        | KA   |
| state_in_kl    | in            | Kerala                           | KL   |
| state_in_ld    | in            | Lakshadweep                      | LD   |
| state_in_mp    | in            | Madhya Pradesh                   | MP   |
| state_in_mh    | in            | Maharashtra                      | MH   |
| state_in_mn    | in            | Manipur                          | MN   |
| state_in_ml    | in            | Meghalaya                        | ML   |
| state_in_mz    | in            | Mizoram                          | MZ   |
| state_in_nl    | in            | Nagaland                         | NL   |
| state_in_or    | in            | Orissa                           | OR   |
| state_in_py    | in            | Puducherry                       | PY   |
| state_in_pb    | in            | Punjab                           | PB   |
| state_in_rj    | in            | Rajasthan                        | RJ   |
| state_in_sk    | in            | Sikkim                           | SK   |
| state_in_tn    | in            | Tamil Nadu                       | TN   |
| state_in_ts    | in            | Telangana                        | TS   |
| state_in_tr    | in            | Tripura                          | TR   |
| state_in_up    | in            | Uttar Pradesh                    | UP   |
| state_in_uk    | in            | Uttarakhand                      | UK   |
| state_in_wb    | in            | West Bengal                      | WB   |
| state_id_ac    | id            | Aceh                             | AC   |
| state_id_ba    | id            | Bali                             | BA   |
| state_id_bb    | id            | Bangka Belitung                  | BB   |
| state_id_bt    | id            | Banten                           | BT   |
| state_id_be    | id            | Bengkulu                         | BE   |
| state_id_go    | id            | Gorontalo                        | GO   |
| state_id_jk    | id            | Jakarta                          | JK   |
| state_id_ja    | id            | Jambi                            | JA   |
| state_id_jb    | id            | Jawa Barat                       | JB   |
| state_id_jt    | id            | Jawa Tengah                      | JT   |
| state_id_ji    | id            | Jawa Timur                       | JI   |
| state_id_kb    | id            | Kalimantan Barat                 | KB   |
| state_id_ks    | id            | Kalimantan Selatan               | KS   |
| state_id_kt    | id            | Kalimantan Tengah                | KT   |
| state_id_ki    | id            | Kalimantan Timur                 | KI   |
| state_id_ku    | id            | Kalimantan Utara                 | KU   |
| state_id_kr    | id            | Kepulauan Riau                   | KR   |
| state_id_la    | id            | Lampung                          | LA   |
| state_id_ma    | id            | Maluku                           | MA   |
| state_id_mu    | id            | Maluku Utara                     | MU   |
| state_id_nb    | id            | Nusa Tenggara Barat              | NB   |
| state_id_nt    | id            | Nusa Tenggara Timur              | NT   |
| state_id_pa    | id            | Papua                            | PA   |
| state_id_pb    | id            | Papua Barat                      | PB   |
| state_id_ri    | id            | Riau                             | RI   |
| state_id_sr    | id            | Sulawesi Barat                   | SR   |
| state_id_sn    | id            | Sulawesi Selatan                 | SN   |
| state_id_st    | id            | Sulawesi Tengah                  | ST   |
| state_id_sg    | id            | Sulawesi Tenggara                | SG   |
| state_id_sa    | id            | Sulawesi Utara                   | SA   |
| state_id_sb    | id            | Sumatra Barat                    | SB   |
| state_id_ss    | id            | Sumatra Selatan                  | SS   |
| state_id_su    | id            | Sumatra Utara                    | SU   |
| state_id_yo    | id            | Yogyakarta                       | YO   |

对于每一行（记录）： 

- 第一列是 的 [外部 ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)要创建的记录 或  更新 
- 第二列是 的 [外部 ID ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)要链接的国家/地区对象  to（国家对象必须事先定义） 
- 第三列是 `name`字段为 `res.country.state`
- 第四列是 `code`字段为 `res.country.state`



>来源 https://www.odoo.com/documentation/10.0/reference/data.html

​                                                

