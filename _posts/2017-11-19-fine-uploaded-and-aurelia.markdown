---
layout: post
title:  "Aurelia and Fine Uploader Drag and Drop"
date:   2017-11-29 13:45:00 +0100
categories: aurelia fine-uploaded
cover: https://cdn.colorlib.com/wp/wp-content/uploads/sites/2/jquery-progress-bars.png
---



In this post we’ll talk about how to integrate and install [Fine Uploader](https://docs.fineuploader.com/branch/master/) (drag and drop) with [Aurelia](http://aurelia.io) js.

For those who don’t know fine uploader is an all in one JavaScript library for uploading files. Fine uploader has been developed in a modular way allowing a developer to pick a choose which components they need. The project is split into Core, UI and drag and drop. I will not be covering the UI component of the library instead I will discuss the core api and the drag and drop components and I’ll create a custom UI.

I will assume if you’re reading this you know what Aurelia is. I will be installing fine uploader to a new project created by the aurelia-cli.

First we’ll need to install the component:


`npm install fine-uploader —save`


This will download the following files:

![folder structure](/assets/images/fine-uploader-folder-structure.png)

Next we need to tell the Aurelia confit about our new component. In your aurelia_config.json add:

{% highlight json linenos=table %}
{
    "name": "fine-uploader",
    "path": "../node_modules/fine-uploader/fine-uploader",
    "main": "fine-uploader.core.js"
},
{
    "name": "fine-uploader-dnd",
    "path": "../node_modules/fine-uploader/dnd",
    "main": "dnd.js"
}
{% endhighlight %} 

This will tel Aurelia that we have a new module called fine-uploader and the files to bundle into our app.bundle.js are located at (file path)

Now we have a module we can import. We now have two options when integrating with Aurelia. 1. A new custom component or 2. A new custom attribute. In my opinion the attribute is more developer friendly as we can just append the attribute to an element and extends its functionality.

{% highlight ts linenos=table %}
import { UploadService } from "./../../services/upload-service";
import { FileTypeService } from "./../../services/file-type-service";
import { autoinject } from "aurelia-framework";
import * as qq2 from "fine-uploader-dnd";
import { EventAggregator } from "aurelia-event-aggregator";
import { UploadProgress } from "../../models/upload-progress";
import { EventNames } from "../../events/event-name";

@autoinject
export class DragAndDropUploadCustomAttribute {
    public activeClass: string = "drag-and-drop-upload__active";
    public enabled: boolean = false;

    private _model: DirectoryEntity;
    private _dragAndDrop: any;

    private _addActiveClassListner: any = this.addActiveClass.bind(this);
    private _removeActiveClassListner: any = this.removeActiveClass.bind(this);

    constructor(
        private element: Element,
        private ea: EventAggregator,
        private uploaderService: UploadService) {
    }

    public valueChanged(newValue: DirectoryEntity) {
        this._model = newValue;

        if (!this._model) return;

        if (this._model.type === DirectoryEntityType.Folder) {
            if (!this._dragAndDrop) {
                this.init();
            }
        } else if (!!this._dragAndDrop) {
            this.dispose();
        }
    }

    public init() {
        this._dragAndDrop = new (<any>qq2).DragAndDrop({
            dropZoneElements: [this.element],
            classes: {
                dropActive: this.activeClass
            },
            callbacks: {
                processingDroppedFilesComplete: (files: any[], dropTarget) => {
                    if (!this.enabled) {
                        return;
                    }

                    if (!this._model.id) {
                        throw Error("drag-and-drop-upload must be supplied a directory entity Id");
                    }

                    if (files.length === 0) {
                        let progress = new UploadProgress();
                        progress.id = this._model.id;
                        progress.fileName = "EMPTY FOLDER or NOT SUPPORTED";
                        progress.error = "This folder has no files in it or your browser cannot upload folders (we recommend using Google Chrome)";
                        progress.type = DirectoryEntityType.Folder;

                        this.ea.publish(EventNames.DirectoryEntity_InvalidFileUploadAttempted, progress);
                    } else {
                        this.uploaderService.addFiles(files, this._model.id);
                    }
                }
            }
        });

        this.addEvents();
    }

    public detached() {
        this.dispose();
    }

    private dispose() {
        if (!!this._dragAndDrop) {
            this._dragAndDrop.dispose();
            this._dragAndDrop.removeDropzone();

            this.removeEvents();

            delete this._dragAndDrop;
        }
    }

    private addEvents() {
        this.element.addEventListener("dragenter", this._addActiveClassListner);
        this.element.addEventListener("drop", this._removeActiveClassListner);
    }

    private removeEvents() {
        this.element.removeEventListener("dragenter", this._addActiveClassListner);
        this.element.removeEventListener("drop", this._removeActiveClassListner);
    }

    private addActiveClass(e: Event) {
        if (!this.enabled) {
            this.element.classList.remove(this.activeClass);
            return;
        }

        let nodes = document.querySelectorAll("." + this.activeClass);
        for (let index = 0; index < nodes.length; index++) {
            let element = nodes[index];
            if (!!element && this.element !== element) {
                element.classList.remove(this.activeClass);
            }
        }
        this.element.classList.add(this.activeClass);
    }

    private removeActiveClass(e: Event) {
        this.element.classList.remove(this.activeClass);
    }
}
{% endhighlight %} 

Add a new file for typescript support.

{% highlight ts linenos=table %}
/// <reference path="../node_modules/fine-uploader/typescript/fine-uploader.d.ts" />

declare module 'fine-uploader' {
    export = qq;
}

declare module 'fine-uploader-dnd' {
    export = qq;
}
{% endhighlight %} 

I have split the functionality of the actual upload into another object/class.

{% highlight ts linenos=table %}
import { autoinject } from "aurelia-framework";
import { EventAggregator } from "aurelia-event-aggregator";
import { EventNames } from "./../events/event-name";
import { UploadProgress } from "../models/upload-progress";
import * as qq from "fine-uploader";

@autoinject
export class UploadService {

    private static _uploader: FineUploader.qq;
    private static _maxFileSize: number = 1073741824; // 1Gb

    constructor(
        private authService: AuthenticationService,
        private configurationService: ConfigurationService,
        private ea: EventAggregator,
        private fileTypeService: FileTypeService,
        private dialogService: DialogService,
        private dialogHelper: DialogHelper,
        private api: DirectoryEntityClient,
        private folderApi: DirectoryEntityFoldersClient,
        private sizeConverterService: SizeConverterService
    ) {

        if (!UploadService._uploader) {
            UploadService._uploader = new qq.FineUploaderBasic({
                maxConnections: 1,
                request: {
                    endpoint: "/",
                    customHeaders: {}
                },
                messages: {
                    emptyError: "This file is empty and cannot be uploaded"
                },
                callbacks: {
                    onSubmit: (id, name) => {

                        let file = UploadService._uploader.getFile(id);

                        let endpoint = "http://mywebsiteapi.com/url_to_post_to";
                        UploadService._uploader.setEndpoint(endpoint, id);

                        UploadService._uploader.setParams({
                            // not required but i prefer a variable of a known name. I don't want this breaking if fine uploader ever changes their default name
                            filename: file.name, 
                        }, id);

                        let progress = new UploadProgress();
                        progress.id = id;
                        progress.fileName = name;
                        progress.totalBytes = 0;
                        progress.uploadedBytes = 0;

                        this.ea.publish(EventNames.DirectoryEntity_NewFileUploadStarted, progress);
                    },
                    onComplete: (id, name, fileUploadResponse: CreateResponse) => {
                        if (fileUploadResponse.success) {
                            let file = UploadService._uploader.getFile(id);
                            this.ea.publish(EventNames.DirectoryEntity_NewOrUpdated, file.parentId);
                        }

                        let progress = new UploadProgress();
                        progress.id = id;
                        progress.fileName = fileUploadResponse.name;
                        progress.error = fileUploadResponse.error;

                        this.ea.publish(EventNames.DirectoryEntity_NewFileUploadComplete, progress);
                    },
                    onProgress: (id, name, uploadedBytes, totalBytes) => {
                        let progress = new UploadProgress();
                        progress.id = id;
                        progress.fileName = name;
                        progress.totalBytes = totalBytes;
                        progress.uploadedBytes = uploadedBytes;

                        this.ea.publish(EventNames.DirectoryEntity_NewFileUploadProgress, progress);
                    },
                    onError: (id, name, message) => {
                        let progress = new UploadProgress();
                        progress.id = id;
                        progress.fileName = name;
                        progress.error = message;

                        this.ea.publish(EventNames.DirectoryEntity_InvalidFileUploadAttempted, progress);
                    }
                }
            });
        }
    }

    public async addFiles(files: QQFile[], parentId: number) {
        UploadService._uploader.addFiles(filesForUpload);
    }
}
{% endhighlight %} 

This allows us to reuse this code is we want to allow a user to upload via a browse button. I’ll show how this is possible in [another post]({% post_url 2017-11-19-fine-uploaded-and-aurelia-browser-button %})
