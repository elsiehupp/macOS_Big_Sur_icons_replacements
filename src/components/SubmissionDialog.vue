<template>
  <coral-dialog id="submissionDialog">

    <coral-dialog-header>
      Submit an icon
      
      <p class="coral-Body--M p-t-8 m-b-4">
        Visit the <router-link coral-close="" to="/resources" class="coral-Link">resources page </router-link> for icon templates.
      </p>

      <p class="coral-Body--M p-t-4 p-b-4 m-b-0">
          All icons must be .png images and have a resolution of 1024px x 1024px.
        <br>
        <span class="coral-Body--S m-b-0 p-t-0 opacity-50">
          Support for .icns is on my (long) todo list.
        </span>
      </p>


    </coral-dialog-header>
    
    <coral-dialog-content class="coral-dialog-content" style="height: 100%">

      <div v-if="isLoading" class="loading-overlay">
        <div class="loading-popup">
          <coral-progress indeterminate>{{ uploadProgress }}/{{ totalNumFiles+1 }} icons uploaded</coral-progress>
        </div>
      </div>

      <div class="icon-upload-grid ">
        
        <!-- Initial Drag to upload -->
        <coral-fileupload
          v-if="!imageData"
          name="file"
          @change="selectIcon"
          class="m-auto fileupload-wrapper"
          accept="image/png" multiple
        >
          <div
            coral-fileupload-select=""
            coral-fileupload-dropzone=""
          >
            <div class="fileUpload-dropZone drop-zone"> 
              <div class="h-full">
                <div class="drop-zone-wrapper">
                  <coral-icon class="m-auto" :icon="coralIcons.addIcon" size="XL" alt="Larger" title="XL"></coral-icon>
                  <span class="m-auto"> Add/drop files </span>
                </div>
              </div>
            </div>
          </div>
        </coral-fileupload>
        
        <coral-fileupload
          v-if="imageData"
          name="file"
          @change="selectIcon"
          class="m-auto fileupload-wrapper hidden-fileuploader"
          accept="image/png" multiple
        >
          
          <div
            coral-fileupload-dropzone=""
          >
            <div class="fileUpload-dropZone drop-zone drop-zone-hidden"> 
              <div class="h-full">
                
                <div v-if="imageData" class="upload-card-wrapper">
                
                  <div v-for="icon in filesToShow" :key="icon.randId" class="upload-card coral-Well">
                    <!-- {{icon}} -->
                    <SubmissionIconPreview
                      :icon="icon"
                      :removeFile="removeFile"
                    />

                    <form class="coral-FormGroup m-0 p-l-4" style="width: calc(100% - 5px)">
                      
                      <!-- AppName -->
                      <div class="coral-FormGroup-item">
                        <label :id="'appNameLabel'+icon.randId" class="coral-FieldLabel">
                          App name
                        </label>
                        <input
                          is="coral-textfield"
                          class="coral-Form-field"
                          type="text"
                          required=""
                          :id="'appNameUploadField'+icon.randId"
                          :labelledby="'appNameLabel'+icon.randId"
                          :value="icon.name"
                          v-on:change="getValue($event, icon.randId, 'name')"
                        >
                      </div>
                      
                      <!-- App category -->
                      <div class="coral-FormGroup-item">
                        <label :id="'categoryUploadLabel'+icon.randId" class="coral-FieldLabel">
                          App category
                        </label>
                        <div class="dropdown-select-chevron relative">
                          <select
                            name="categoryUploadField"
                            :id="'categoryUploadField'+icon.randId"
                            placeholder="Select category"
                            class="dropdown-select"
                            v-on:change="getValue($event, icon.randId, 'category')"
                          >
                            <option
                              value=""
                              disabled selected
                            >
                              Select category (required)
                            </option>
                            <option
                              v-for="category in getAppCategories"
                              :key="category.name+icon.randId+Math.floor(Math.random() * 10000000 + 1)"
                              :id="icon.randId+category.id"
                              :value="category.id"
                              :selected="icon.category.includes(category.id)"
                            >
                              {{ category.name }}
                            </option>
                          </select>
                        </div>
                      </div>
                      
                      <!-- Type of icon -->
                      <div class="coral-FormGroup-item">
                        <label :id="'TypeUploadLabel'+icon.randId" class="coral-FieldLabel">
                          Type of icon
                        </label>
                        <div class="dropdown-select-chevron relative">
                          <select
                            name="TypeUploadField"
                            :id="'TypeUploadField'+icon.randId"
                            placeholder="Select Type"
                            class="dropdown-select"
                            v-on:change="getValue($event, icon.randId, 'type')"
                          >
                            <option
                              v-for="type in getIconType"
                              :key="type.name+icon.randId+Math.floor(Math.random() * 10000000 + 1)"
                              :value="type.id"
                              :selected="selectedOption(type.id, icon.type)"
                            >
                              {{ type.name }}
                            </option>
                          </select>
                        </div>
                      </div>
                      
                      <!-- App website -->
                      <div class="coral-FormGroup-item">
                        <label :id="'appWebsiteUploadLabel'+icon.randId" class="coral-FieldLabel">
                          App website (optional)
                        </label>
                        <input
                          :id="'appWebsiteUploadField'+icon.randId"
                          is="coral-textfield"
                          :labelledby="'appWebsiteUploadLabel'+icon.randId"
                          class="coral-Form-field"
                          type="url"
                          placeholder="The app's developer website"
                          v-on:change="getValue($event, icon.randId, 'appWebsite')"
                        >
                      </div>
                      
                      <!-- .icns file -->
                      <div class="coral-FormGroup-item">
                        <div>
                          <label
                            class="coral-FieldLabel"
                            for="icnsFileUpload"
                          >
                            .icns file (optional)
                          </label>

                          <button
                            is="coral-button"
                            for="icnsFileUpload"
                            @click="openFileUpload('icnsFileUpload', icon.randId)"
                          >
                            <span v-if="!filesToShow[icon.randId].icnsFile">
                              Upload .icns file
                            </span>
                            <span v-if="filesToShow[icon.randId].icnsFile">
                              Remove .icns file
                            </span>
                          </button>
                          <input
                            type="file"
                            id="icnsFileUpload"
                            accept=".icns"
                            class="hidden"
                            @change="setIcns($event, icon.randId)"
                          >
                        </div>
                      </div>
                      
                      <!-- Is dark mode -->
                      <div class="coral-FormGroup-item">
                        <coral-checkbox
                          :id="'isDarkUpload'+icon.randId"
                          v-on:change="getCheckedValue($event, icon.randId, 'isDarkMode')"
                        >
                          Is dark mode
                        </coral-checkbox>
                        <coral-checkbox
                          :id="'isAuthorUpload'+icon.randId"
                          v-on:change="getCheckedValue($event, icon.randId, 'isAuthor')"
                        >
                          * I'm the author of this icon
                        </coral-checkbox>
                      </div>

                    </form>
                  </div>
                  
                  <!-- Drag to upload -->
                  <div
                  class="m-auto fileupload-wrapper"
                    coral-fileupload-select=""
                  >
                    <div class="fileUpload-dropZone drop-zone">
                      <div class="h-full">
                        <div class="drop-zone-wrapper">
                          <coral-icon class="m-auto" :icon="coralIcons.addIcon" size="XL" alt="Larger" title="XL"></coral-icon>
                          <span class="m-auto"> Add/drop files </span>
                        </div>
                      </div>
                    </div>
                  </div>

                </div>

              </div>
            </div>
          </div>

        </coral-fileupload>
        
      </div>
    </coral-dialog-content>
    
    <coral-dialog-footer>
      <button is="coral-button" coral-close="">Cancel</button>
      <button is="coral-button" variant="cta" v-if="validateForm" @click="onUpload">Upload</button>
      <button is="coral-button" variant="cta" v-else disabled="" >Upload</button>
      <!-- <button v-if="imageData && email != '' " is="coral-button" variant="cta" @click="onUpload">Upload</button> -->
      <!-- <button v-if="!imageData || email == '' " is="coral-button" disabled>Upload</button> -->
    </coral-dialog-footer>

  </coral-dialog>
</template>

<script>
import { defineAsyncComponent } from 'vue'
import { mapActions, mapGetters } from 'vuex';
import Parse from 'parse/dist/parse.min.js';

import deleteIcon from "../assets/icons/delete.svg"
import addCoralIcon from "../assets/icons/add.svg"
import iconTemplate from "../assets/icons/icon_template.png"
import iconBrew from "../api/iconBrew.js"

export default {
    name:"SubmissionDialog",

    props:{
    },

    components:{
      "SubmissionIconPreview": defineAsyncComponent(() => import('@/components/SubmissionIconPreview.vue'))
    },

    data(){
      return{
        imageData: false,
        validUpload: false,
        filesToShow: {},
        filesToUpload: {},
        coralIcons:{
          addIcon: addCoralIcon,
          deleteIcon: deleteIcon,
        },
        isCheckingSize: false,
        iconTemplate: iconTemplate,
        uploadProgress: 0,
        totalNumFiles: 0,
        email: "",
        credit: "",
        appName: "",
        yourName: "",
        isLoading: false,
        // categoriesList: [{CategoryName: "test"}, {CategoryName: "Test 2"}]
        categoriesList: []
      }
    },

    methods:{
      ...mapActions(['showToast', 'fetchAppCategories', 'fetchIconType']),

      openFileUpload(uploadId, iconId){
        event.preventDefault()
        const uploadInput = document.getElementById(uploadId)
        let iconFile = this.filesToShow[iconId].icnsFile

        if (iconFile) {
          this.filesToShow[iconId].icnsFile = ''
          uploadInput.value = '';
        }else{
          uploadInput.click();
        }
      },

      iconBrew(icon, size){
        console.log(iconBrew[icon + size]);
        return iconBrew[icon + size];
      },

      checkSize(id){
        this.isCheckingSize = !this.isCheckingSize;
        document.getElementById(id).classList.toggle("check-size");
      },

      getCheckedValue(e, appName, field){
        let parent = this
        let fieldValue = e.target.checked
        parent.filesToShow[appName][field] = fieldValue
      },

      getValue(e, appName, field){
        console.log(e);
        let target = e.target
        let fieldValue = target.value
        this.filesToShow[appName][field] = fieldValue;
      },

      selectedOption(option, value){
        if (option == value) {
          return true
        } else {
          return false
        }
      },

      removeFile(e, randId){
        delete this.filesToUpload[randId]
        delete this.filesToShow[randId]

        // If imageURL is empty, show the upload files component
        if (Object.keys(this.filesToShow).length === 0) {
          parent.imageData = false
        }
      },

      setIcns(event, iconId){
        let file = event.target.files[0]
        this.filesToShow[iconId].icnsFile = file
        console.log("event: ", this.filesToShow[iconId]);
      },

      selectIcon(event) {
        // Get selected image
        let parent = this
        let files = event.target.uploadQueue
        
        // Only run if what has triggered the change is the upload wrapper
        if (!event.target.classList.contains("fileupload-wrapper")) {
          return
        } else{
        // // Go through all the files that have been selected
          for(let fileNum in files){
            let file = files[fileNum].file
            const objectURL = window.URL.createObjectURL(file);

            // Set image to new Image to get width and height
            var img = new Image();
            img.onload = function() {
              let width = this.width
              let height = this.height
              if (width, height != 1024 ) {
                window.URL.revokeObjectURL(this.src);
                parent.showToast({
                  id: "toastMessage",
                  message: "Icon needs to be 1024px, yours is "+height+"px x "+width+"px",
                  variant: "error"
                })
              } else{
                  let fileName = file.name.replace('.png', '')
                  let randId = Math.floor(Math.random() * 10000000 + 1)+"-"+fileName+Object.keys(parent.filesToShow).length
                  parent.filesToUpload[randId] = file
                  
                  let value = {
                    img: objectURL,
                    name: fileName,
                    file: file,
                    isDarkMode: false,
                    category: "",
                    appWebsite: "",
                    type: "Zz9QX1BBIZ",
                    randId: randId,
                  }

                  parent.imageData = true
                  parent.filesToShow[randId] = value;
                  // parent.$set(parent.filesToShow, randId, value)
              }

            }
            // Create URL of file to dislay back the image
            img.src = objectURL;
          }

          parent.totalNumFiles = Object.keys(parent.filesToShow).length
          event.target.clear()
        }

      },

      async onUpload(){
        let parent = this
        let DownloadCount = Parse.Object.extend("DownloadCount")
        let IconType = Parse.Object.extend("IconType")
        let Categories = Parse.Object.extend("Categories")
        let typQuery = new Parse.Query(IconType)
        let categoryQuery = new Parse.Query(Categories)

        // Get today's date
        var today = new Date();
        var dd = String(today.getDate()).padStart(2, '0');
        var mm = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
        var yyyy = today.getFullYear();

        today = dd + '/' + mm + '/' + yyyy;

        window.plausible("IconSubmission", {props: { date: today}})

        parent.isLoading = true
        let dialog = document.getElementById('submissionDialog')

        for(let fileNum in parent.filesToShow){
          let file =  parent.filesToShow[fileNum].file;
          let appName = parent.filesToShow[fileNum].name;
          let randId = parent.filesToShow[fileNum].randId;
          let typeId = parent.filesToShow[fileNum].type;
          let isDarkMode = parent.filesToShow[fileNum].isDarkMode;
          let isAuthor = parent.filesToShow[fileNum].isAuthor;
          let icnsFile = parent.filesToShow[fileNum].icnsFile ? parent.filesToShow[fileNum].icnsFile: '';
          let icnsFileUrl = '';
          let parseIcnsFile;
          
          // Retrieve Category Parse object
          let category = parent.filesToShow[fileNum].category;
          categoryQuery.get(category)
          category = await categoryQuery.find();
          category = category[0];
          
          var type 
          // Retrieve IconType Parse object
          for(let item in parent.getIconType){
            if (parent.getIconType[item].id == typeId) {
              type = parent.getIconType[item].parseObj
            }
          }

          let fileName;

          if (/^[A-Za-z][A-Za-z0-9]*$/.test(appName)) {
            fileName = appName
          } else {
            let d = new Date()
            fileName = Math.round(Math.random()*10000 + d.getTime() )
            fileName = fileName.toString()
          }
          
          const Icons = Parse.Object.extend("Icons2");
          const icons = new Icons()

          // const parseFile = new Parse.File(fileName, file); // Set file to new Parse object
          console.log("icnsFile: ", icnsFile);
          if (icnsFile != '') {
            parseIcnsFile = new Parse.File(fileName+'.icns', icnsFile); // Set file to new Parse object
            let saveIcns = await parseIcnsFile.save()
            icnsFileUrl = saveIcns._url;
          }

          const parseFile = new Parse.File(fileName, file); // Set file to new Parse object
          parseFile.save().then((uploaded) => {
            console.log("Success: ", uploaded._url);
            let iconUrl = uploaded._url.replace('http:', "https:")
            let currentUser = Parse.User.current()
          
            let dataToStore = {
              appName: appName,
              fileName: fileName,
              highResPngFile: parseFile,
              highResPngUrl: iconUrl,
              timeStamp: Date.now(),
              approved: false,
              user: currentUser,
              email: Parse.User.current().getEmail(),
              usersName: Parse.User.current().getUsername(),
              credit: Parse.User.current().get("credit"),
              category: category,
              type: type,
              DownloadCount: new DownloadCount(),
              isDarkMode: isDarkMode,
              isAuthor: isAuthor
            }

            if (parseIcnsFile != '') {
              dataToStore.icnsFile = parseIcnsFile;
              dataToStore.icnsUrl = icnsFileUrl;
            }

            icons.set(dataToStore);
            
            const acl = new Parse.ACL();
            acl.setPublicReadAccess(true);
            acl.setWriteAccess(Parse.User.current().id, true)
            acl.setRoleWriteAccess("Admin", true);

            icons.setACL(acl);
            icons.save().then((icon) => { // Reset input boxes
              
              // Add icon relationship to user
              let userRelation = currentUser.relation("icons")
              userRelation.add(icons)
              currentUser.save().then().catch((error) =>{
                console.log("error: ", error)
              });

              parent.imageData = {}
              parent.picture= null
              parent.uploadProgress++
              
              delete parent.filesToShow[randId]
              delete parent.filesToUpload[randId]

              if (Object.keys(parent.filesToUpload).length === 0) {
                  parent.isLoading = false
                  parent.imageData = false
                  parent.uploadProgress = 0

                  parent.showToast({
                    id: "toastMessage",
                    message: "All icons have been uploaded.",
                    variant: "success"
                  })
                  dialog.hide()
                }
            },(error)=>{
              console.log("Data NOT saved: ", error);
            })
          }, function(error) {
            console.log("error: ", error);
            parent.isLoading = false
            parent.showToast({
              id: "toastMessage",
              message: "There was an error, get in touch with @elrumo on Twitter",
              variant: "error"
            })
          });

        }
      },

    },

    async mounted(){ 
      this.fetchAppCategories()
      this.fetchIconType()

      // const IconsBase = Parse.Object.extend("Icons2");
      // const query = new Parse.Query(IconsBase);
      // const newQuery = await query.get('Bg7PB3BbSA')

      // const icnsFile = await newQuery.get("icnsFile");
      
      // if(icnsFile){
      //   console.log(icnsFile.url());
      // }

    },

    computed:{
      ...mapGetters(["getAppCategories", "getIconType"]),

      validateForm(){
        let parent = this;
        let filesToShow = parent.filesToShow
        var isValid = []
        
        if (parent.imageData) {
          for (let item in filesToShow){
            let isAuthor = filesToShow[item].isAuthor;
            for (let field in filesToShow[item]){
              let submission = filesToShow[item][field]
              if (submission != "" && submission != undefined && field != "appWebsite" && isAuthor) {
                isValid.push(true)
              } else if (field != "appWebsite" && field != "isDarkMode") {
                isValid.push(false)
              }
            }
          }
        } else {
          isValid.push(false)
        }

        return !isValid.some((el) => { return el == false })
      }
    }

}
</script>

<style>
</style>