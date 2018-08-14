# angular6-schematics
schematics
前情提要:for  Angular 6以上版本，範例為建立一個全新專案，代碼貼上後請自行shift+alt+f排版 
 
1.  
使用指令 ng new myTest  建立一新專案 
 
2.  
專案內使用指令 npm install -g @angular-devkit/schematics-cli 安裝 Schematics 命令列工具 
 
3.  
專案內使用指令 schematics blank --name=my-first-schema 創建一個 Schematics，其名為 my-first-schema 
 
4.  
將 my-first-schema/src 資料夾中的 collection.json 內容改為: 
 
{ 
"$schema": "../node_modules/@angular-devkit/schematics/collection-schema.json", 
"schematics": { 
"my-first-schema": { 
"aliases": ["mfs"], 
"factory": "./my-first-schema", 
"description": "my first schematic.", 
"schema": "./my-first-schema/schema.json" 
} 
} 
} 
 
 

 
備註 1. 
 
5. 
在 my-first-schema/src/my-first-schema新增一檔案，名稱為 schema.json 
 

 
 
 schema.json    內容為: 
 
{ 
"$schema": "http://json-schema.org/schema", 
"id": "my-first-schema", 
"title": "my1er Schema", 
"type": "object", 
"properties": { 
"name": { 
"type": "string", 
"default": "name" 
}, 
"path": { 
"type": "string", 
"default": "app" 
}, 
"appRoot": { 
"type": "string" 
}, 
"sourceDir": { 
"type": "string", 
"default": "src/app" 
}, 
"service": { 
"type": "boolean", 
"default": false, 
"description": "Flag to indicate whether service should be generated.", 
"alias": "vgs" 
} 
} 
} 
 
 
備註2. 
 
6. 
在 my-first-schema/src/my-first-schema新增一檔案，名稱為schema.ts 
 

 
 
schema.ts內容為: 
 
export interface schemaOptions { 
 
 
name: string; 
 
 
appRoot: string; 
 
 
path: string; 
 
 
sourceDir?: string; 
 
 
service: boolean; 
 
 
prefix?: string; 
 
 
selector?: string; 
 
 
} 
 
 
 
7.  
修改 my-first-schema/src/my-first-schema 內的 index.ts 內容為: 
 
import { 
  Rule, 
  SchematicsException, 
  apply, 
  Tree, 
  SchematicContext, 
  branchAndMerge, 
  chain, 
  mergeWith, 
  move, 
  template, 
  url 
} from '@angular-devkit/schematics'; 
import * as stringUtils from '@angular-devkit/core/src/utils/strings'; 
import { schemaOptions as FeatureOptions } from './schema'; 
 
function buildSelector(options: FeatureOptions) { 
  let selector = stringUtils.dasherize(options.name); 
  if (options.prefix) { 
    selector = `${options.prefix}-${selector}`; 
  } 
 
  return selector; 
} 
 
export default function(options: FeatureOptions): Rule { 
  options.selector = options.selector || buildSelector(options); 
  const sourceDir = options.sourceDir; 
  if (!sourceDir) { 
    throw new SchematicsException(`sourceDir option is required.`); 
  } 
 
  return (host: Tree, context: SchematicContext) => { 
    const templateSource = apply(url('./files'), [ 
      template({ 
        ...stringUtils, 
        ...options 
      }), 
      move(sourceDir) 
    ]); 
 
    return chain([branchAndMerge(chain([mergeWith(templateSource)]))])(host, context); 
  }; 
} 
 
8.  
在 my-first-schema/src/my-first-schema 中建立一資料夾名為  files 
 
9.  
在 my-first-schema/src/my-first-schema/files 中新增三個檔案 
 

 
my-first-schema.component.ts 
內容為: 
import { Component, Input, } from '@angular/core'; 
 
 
@Component({ 
selector: 'my-first-schema-component', 
templateUrl: './my-first-schema.component.html', 
styleUrls: [ './my-first-schema.component.css' ] 
}) 
 
 
export class MyFirstSchemaComponent { 
 
 
constructor(){ 
console.log( '<%= classify(name) %>' ); 
} 
 
 
} 
 
my-first-schema.component.html 
內容為: 
<% if (service) { %> 
<h1>Hola Service</h1> 
<% } %> 
 
 
<% if (!service) { %> 
<h1>Hola no Service</h1> 
<% } %> 
 
my-first-schema.component.css 
內容為空 
 
10.  
在 my-first-schema\src\my-first-schema 開啟終端機   輸入指令npm run build 
注意終端機位址有沒有錯，要在src內的my-first-schema   再run build 
 
成功後會多出一堆檔案，紅色圈起的 

 

 
11. 
ng new test-schematics 
 
建立另一新專案來測試 
 
12. 
進入新專案  在node_modules\@schematics\angular貼上先前創建專案中 
my-first-schema\src內的my-first-schema資料夾(整包複製貼上) 

 
複製到新專案貼到node_modules\@schematics\angular內: 

 
13. 
在新專案的  node_modules\@schematics\angular內的collection.json檔案中新增 
"my-first-schema": { 
"aliases": ["mfs"], 
"factory": "./my-first-schema", 
"description": "my first schematic.", 
"schema": "./my-first-schema/schema.json" 
} 
 
14. 
回到專案 使用指令 
ng g my-first-schema mfs  --service  --name="Mfs"  --collection mfs 
新增成功即可在app/src 的資料夾my-schema內看見檔案 
 
 

 
 

 
基本上到這邊已成功 
 
15. 
若要修改模板內容 
就修改新專案中node_modules\@schematics\angular\my-first-schema\files 
Files資料夾內的三個檔案 

 
--------------------------------------------------------------------------------------------------------------------------------------- 
以下為更改路徑與自訂名稱 
因為依照上述做法做完後生成檔案的位置會被釘在同一個地方，所以要再依照以下方法更改 
 
**改原本舊的專案，要測試再複製過去新專案測，因為還有build後生成檔案的問題，下述改完直接build後生成的檔案會一併更新，如果改新專案內的檔案就連index.js也要一併修改 
16. 
更改生成路徑: 
把第5步驟的   schema.json檔案直接全刪改成 
{ 
"$schema": "http://json-schema.org/schema", 
"id": "my-first-schema", 
"title": "my1er Schema", 
"type": "object", 
"properties": { 
"name": { 
"type": "string", 
"default": "name" 
}, 
"path": { 
"type": "string", 
"format": "path", 
"description": "The path to create the component.", 
"visible": false 
}, 
"appRoot": { 
"type": "string" 
}, 
"sourceDir": { 
"type": "string", 
"default": "src/app" 
}, 
"service": { 
"type": "boolean", 
"default": false, 
"description": "Flag to indicate whether service should be generated.", 
"alias": "vgs" 
} 
} 
} 
 
在這裡主要是更改了path的設定‧ 
然後再到第7步驟的index.ts   

或者複製貼上以下 
 
import { 
Rule, 
SchematicsException, 
apply, 
Tree, 
SchematicContext, 
branchAndMerge, 
chain, 
mergeWith, 
move, 
template, 
url 
} from '@angular-devkit/schematics'; 
import * as stringUtils from '@angular-devkit/core/src/utils/strings'; 
import { schemaOptions as FeatureOptions } from './schema'; 
 
 
function buildSelector(options: FeatureOptions) { 
let selector = stringUtils.dasherize(options.name); 
if (options.prefix) { 
selector = `${options.prefix}-${selector}`; 
} 
 
 
return selector; 
} 
 
 
export default function(options: FeatureOptions): Rule { 
options.selector = options.selector || buildSelector(options); 
const sourceDir = options.path; 
if (!sourceDir) { 
throw new SchematicsException(`sourceDir option is required.`); 
} 
 
 
return (host: Tree, context: SchematicContext) => { 
const templateSource = apply(url('./files'), [ 
template({ 
...stringUtils, 
...options 
}), 
move(sourceDir) 
]); 
 
 
return chain([branchAndMerge(chain([mergeWith(templateSource)]))])(host, context); 
}; 
} 
 
 
然後再改 schema.ts 
 
export interface schemaOptions { 
 
 
name: string; 
 
 
appRoot: string; 
 
 
path: string; 
 
 
sourceDir?: string; 
 
 
service: boolean; 
 
 
prefix?: string; 
 
 
selector?: string; 
 
 
/** 
* The name of the project. 
*/ 
project?: string; 
/** 
* Specifies if the style will be in the ts file. 
*/ 
inlineStyle?: boolean; 
/** 
* Specifies if the template will be in the ts file. 
*/ 
inlineTemplate?: boolean; 
/** 
* Specifies the view encapsulation strategy. 
*/ 
viewEncapsulation?: ('Emulated' | 'Native' | 'None'); 
/** 
* Specifies the change detection strategy. 
*/ 
changeDetection?: ('Default' | 'OnPush'); 
/** 
* The file extension to be used for style files. 
*/ 
styleext?: string; 
/** 
* Specifies if a spec file is generated. 
*/ 
spec?: boolean; 
/** 
* Flag to indicate if a dir is created. 
*/ 
flat?: boolean; 
/** 
* Flag to skip the module import. 
*/ 
skipImport?: boolean; 
/** 
* Allows specification of the declaring module. 
*/ 
module?: string; 
/** 
* Specifies if declaring module exports the component. 
*/ 
export?: boolean; 
} 
 
 
 
 
 
 
 
 
路徑修改完成後記得再  run  build一次再把舊專案的資料夾複製到新專案 
 
17. 
自行設定名稱 component的設定 與更新module 
原本的名稱會被固定叫做my-first-schema 
這裡要修改新專案內的內容 
 
在剛放入新專案node_modules\@schematics\angular\my-first-schema\files內建立 
__name@dasherize@if-flat__ 新資料夾 
然後把原本的三個檔案改名後移入新資料夾中 
 

 
然後修改  __name@dasherize__.component.ts 內容 
 
import { Component, OnInit<% if(!!viewEncapsulation) { %>, ViewEncapsulation<% }%><% if(changeDetection !== 'Default') { %>, ChangeDetectionStrategy<% }%> } from '@angular/core'; 
 
 
@Component({ 
selector: '<%= selector %>',<% if(inlineTemplate) { %> 
template: ` 
<p> 
<%= dasherize(name) %> works! 
</p> 
`,<% } else { %> 
templateUrl: './<%= dasherize(name) %>.component.html',<% } if(inlineStyle) { %> 
styles: []<% } else { %> 
styleUrls: ['./<%= dasherize(name) %>.component.<%= styleext %>']<% } %><% if(!!viewEncapsulation) { %>, 
encapsulation: ViewEncapsulation.<%= viewEncapsulation %><% } if (changeDetection !== 'Default') { %>, 
changeDetection: ChangeDetectionStrategy.<%= changeDetection %><% } %> 
}) 
export class <%= classify(name) %>Component implements OnInit { 
 
 
constructor() { } 
 
 
ngOnInit() { 
} 
 
 
} 
 
 
再來打開此 index.js 

 
全刪貼上 
"use strict"; 
Object.defineProperty(exports, "__esModule", { value: true }); 
/** 
* @license 
* Copyright Google Inc. All Rights Reserved. 
* 
* Use of this source code is governed by an MIT-style license that can be 
* found in the LICENSE file at https://angular.io/license 
*/ 
const core_1 = require("@angular-devkit/core"); 
const schematics_1 = require("@angular-devkit/schematics"); 
const ts = require("typescript"); 
const ast_utils_1 = require("../utility/ast-utils"); 
const change_1 = require("../utility/change"); 
const config_1 = require("../utility/config"); 
const find_module_1 = require("../utility/find-module"); 
const parse_name_1 = require("../utility/parse-name"); 
const project_1 = require("../utility/project"); 
const validation_1 = require("../utility/validation"); 
function addDeclarationToNgModule(options) { 
return (host) => { 
if (options.skipImport || !options.module) { 
return host; 
} 
const modulePath = options.module; 
const text = host.read(modulePath); 
if (text === null) { 
throw new schematics_1.SchematicsException(`File ${modulePath} does not exist.`); 
} 
const sourceText = text.toString('utf-8'); 
const source = ts.createSourceFile(modulePath, sourceText, ts.ScriptTarget.Latest, true); 
const componentPath = `/${options.path}/` 
+ (options.flat ? '' : core_1.strings.dasherize(options.name) + '/') 
+ core_1.strings.dasherize(options.name) 
+ '.component'; 
const relativePath = find_module_1.buildRelativePath(modulePath, componentPath); 
const classifiedName = core_1.strings.classify(`${options.name}Component`); 
const declarationChanges = ast_utils_1.addDeclarationToModule(source, modulePath, classifiedName, relativePath); 
const declarationRecorder = host.beginUpdate(modulePath); 
for (const change of declarationChanges) { 
if (change instanceof change_1.InsertChange) { 
declarationRecorder.insertLeft(change.pos, change.toAdd); 
} 
} 
host.commitUpdate(declarationRecorder); 
if (options.export) { 
// Need to refresh the AST because we overwrote the file in the host. 
const text = host.read(modulePath); 
if (text === null) { 
throw new schematics_1.SchematicsException(`File ${modulePath} does not exist.`); 
} 
const sourceText = text.toString('utf-8'); 
const source = ts.createSourceFile(modulePath, sourceText, ts.ScriptTarget.Latest, true); 
const exportRecorder = host.beginUpdate(modulePath); 
const exportChanges = ast_utils_1.addExportToModule(source, modulePath, core_1.strings.classify(`${options.name}Component`), relativePath); 
for (const change of exportChanges) { 
if (change instanceof change_1.InsertChange) { 
exportRecorder.insertLeft(change.pos, change.toAdd); 
} 
} 
host.commitUpdate(exportRecorder); 
} 
return host; 
}; 
} 
function buildSelector(options, projectPrefix) { 
let selector = core_1.strings.dasherize(options.name); 
if (options.prefix) { 
selector = `${options.prefix}-${selector}`; 
} 
else if (options.prefix === undefined && projectPrefix) { 
selector = `${projectPrefix}-${selector}`; 
} 
return selector; 
} 
function default_1(options) { 
return (host, context) => { 
const workspace = config_1.getWorkspace(host); 
if (!options.project) { 
throw new schematics_1.SchematicsException('Option (project) is required.'); 
} 
const project = workspace.projects[options.project]; 
if (options.path === undefined) { 
options.path = project_1.buildDefaultPath(project); 
} 
options.module = find_module_1.findModuleFromOptions(host, options); 
const parsedPath = parse_name_1.parseName(options.path, options.name); 
options.name = parsedPath.name; 
options.path = parsedPath.path; 
options.selector = options.selector || buildSelector(options, project.prefix); 
validation_1.validateName(options.name); 
validation_1.validateHtmlSelector(options.selector); 
const templateSource = schematics_1.apply(schematics_1.url('./files'), [ 
options.spec ? schematics_1.noop() : schematics_1.filter(path => !path.endsWith('.spec.ts')), 
options.inlineStyle ? schematics_1.filter(path => !path.endsWith('.__styleext__')) : schematics_1.noop(), 
options.inlineTemplate ? schematics_1.filter(path => !path.endsWith('.html')) : schematics_1.noop(), 
schematics_1.template(Object.assign({}, core_1.strings, { 'if-flat': (s) => options.flat ? '' : s }, options)), 
schematics_1.move(parsedPath.path), 
]); 
return schematics_1.chain([ 
schematics_1.branchAndMerge(schematics_1.chain([ 
addDeclarationToNgModule(options), 
schematics_1.mergeWith(templateSource), 
])), 
])(host, context); 
}; 
} 
exports.default = default_1; 
 
 
 
 
再開啟 schema.json 
 

 
全刪複製貼上 
{ 
"$schema": "http://json-schema.org/schema", 
"id": "my-first-schema", 
"title": "my1er Schema", 
"type": "object", 
"properties": { 
"name": { 
"type": "string", 
"description": "The name of the component.", 
"$default": { 
"$source": "argv", 
"index": 0 
} 
}, 
"path": { 
"type": "string", 
"format": "path", 
"description": "The path to create the component.", 
"visible": false 
}, 
"appRoot": { 
"type": "string" 
}, 
"sourceDir": { 
"type": "string", 
"default": "src/app" 
}, 
"service": { 
"type": "boolean", 
"default": false, 
"description": "Flag to indicate whether service should be generated.", 
"alias": "vgs" 
}, 
 
 
"project": { 
"type": "string", 
"description": "The name of the project.", 
"$default": { 
"$source": "projectName" 
} 
}, 
"inlineStyle": { 
"description": "Specifies if the style will be in the ts file.", 
"type": "boolean", 
"default": false, 
"alias": "s" 
}, 
"inlineTemplate": { 
"description": "Specifies if the template will be in the ts file.", 
"type": "boolean", 
"default": false, 
"alias": "t" 
}, 
"viewEncapsulation": { 
"description": "Specifies the view encapsulation strategy.", 
"enum": ["Emulated", "Native", "None"], 
"type": "string", 
"alias": "v" 
}, 
"changeDetection": { 
"description": "Specifies the change detection strategy.", 
"enum": ["Default", "OnPush"], 
"type": "string", 
"default": "Default", 
"alias": "c" 
}, 
"prefix": { 
"type": "string", 
"description": "The prefix to apply to generated selectors.", 
"alias": "p", 
"oneOf": [{ 
"maxLength": 0 
}, 
{ 
"minLength": 1, 
"format": "html-selector" 
} 
] 
}, 
"styleext": { 
"description": "The file extension to be used for style files.", 
"type": "string", 
"default": "css" 
}, 
"spec": { 
"type": "boolean", 
"description": "Specifies if a spec file is generated.", 
"default": true 
}, 
"flat": { 
"type": "boolean", 
"description": "Flag to indicate if a dir is created.", 
"default": false 
}, 
"skipImport": { 
"type": "boolean", 
"description": "Flag to skip the module import.", 
"default": false 
}, 
"selector": { 
"type": "string", 
"format": "html-selector", 
"description": "The selector to use for the component." 
}, 
"module": { 
"type": "string", 
"description": "Allows specification of the declaring module.", 
"alias": "m" 
}, 
"export": { 
"type": "boolean", 
"default": false, 
"description": "Specifies if declaring module exports the component." 
} 
}, 
"required": [] 
} 
 
 
在想要建立新模板的位址輸入指令 ng  g mfs Name 即可 
 
 
 
------------------------------------------------------------------------------------------------------------------------------ 
 
添加service 
 

 
在 
test-schematics\node_modules\@schematics\angular\my-first-schema\files\__name@dasherize@if-flat__這層建立新資料夾名稱為service 
並在service中加入兩個檔案，檔案如下。 
 
__name@dasherize__.service.spec.ts的檔案 
 
並貼上以下程式碼 
  
 
import { TestBed, inject } from '@angular/core/testing'; 
 
 
import { <%= classify(name) %>Service } from './<%= dasherize(name) %>.service'; 
 
 
describe('<%= classify(name) %>Service', () => { 
beforeEach(() => { 
TestBed.configureTestingModule({ 
providers: [<%= classify(name) %>Service] 
}); 
}); 
 
 
it('should be created', inject([<%= classify(name) %>Service], (service: <%= classify(name) %>Service) => { 
expect(service).toBeTruthy(); 
})); 
}); 
 
 
__name@dasherize__.service.ts的檔案 
 
 
並貼上以下程式碼 
 
import { Injectable } from '@angular/core'; 
 
 
@Injectable({ 
providedIn: 'root' 
}) 
export class <%= classify(name) %>Service { 
 
 
constructor() { } 
} 
 
 
完成後直接在你指定的檔案位置打上 ng mfs Name即可 
 
 
 
-------------------------------------------------------------------------------------------------------------------- 
備註: 
1. 
$schema => 定義該 collection 架構的 url 地址. 
schematics => 這是你的  schematics 定義. 
my-first-schema => 以後使用這個  schematics 的 cli 名稱. 
aliases => 別名. 
factory => 定義代碼. 
Description => 簡單的説明. 
Schema => 你的 schema 的設置. 這個文檔的內容應該如schema.json所示。我們在其中定義了多個自定義的選項，在使用這個 Schematics 的時候，可以通過這些選項來設置生成的內容。 
2. 
這裡可以設置你的 schematics 的命令列選項，類似於在使用 g 來創建一個新的組件的時候，您可以使用一個 --change-detection 的選項。 

 
參考網址: 
https://hk.saowen.com/a/46346e4b7bd25f8922cc97ed1705f67cbbfe8097ee31e48bd6e335d2b23fa8b1 
https://zhuanlan.zhihu.com/p/37452680 
https://blog.kevinyang.net/2018/01/11/angular-create-schematics/ 
 

