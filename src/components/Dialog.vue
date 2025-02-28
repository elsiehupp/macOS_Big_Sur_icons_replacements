<template>
  <coral-dialog id="submitIcon">

    <coral-dialog-header>
      Submit an icon
    </coral-dialog-header>
    
    <coral-dialog-content>
      
      <div class="dialog-text coral-Well m-b-25">
        On the 2nd of April, the database with all the icons was hacked and deleted. Everything has been secured and is now safe to upload icons. Some old icons might still be unavailabe while I work to get as many icons as I can back.
        <br>
        <br>
        To read more about the incident, <router-link to="/blog/hacked"> read the blog post. </router-link>
      </div>

      <div class="dialog-text">
        <b> We'll let you know by email when the icon has been approved. </b>
        <br>
        <ul class="coral-List p-t-8 p-b-8">
          <li class="coral-List-item">Icon submitted must be a .png file</li>
          <li class="coral-List-item">
            Download template from the <router-link coral-close="" to="/resources"> <b> resources page. </b></router-link>
          </li>
          <li class="coral-List-item"> <b> The file name must be the same as the name of the app. </b></li>
        </ul>
      </div>

      <div v-if="isLoading" class="loading-overlay">
        <div class="loading-popup">
          <coral-progress indeterminate>{{ uploadProgress }}/{{ totalNumFiles }} icons uploaded</coral-progress>
        </div>
      </div>

      <div class="icon-upload-grid">
        
        <div v-if="imageData" class="icons-preview-wrapper">
          
          <div v-for="icon in filesToShow" class="icon-preview" :key="icon.name">
            <img :src="icon.img">
            <coral-quickactions placement="center" target="_prev">
              <coral-quickactions-item type="button" @click="removeFile" :id="icon.name" :icon="coralIcons.delete">Remove file</coral-quickactions-item>
            </coral-quickactions>
          </div>

          <coral-fileupload name="file" @change="selectIcon" class="m-auto" accept="image/png" multiple>
            <div coral-fileupload-dropzone=""  class="fileUpload-dropZone drop-zone"> 
              <div class="h-full" coral-fileupload-select="">
                <div class="drop-zone-wrapper">
                  <coral-icon class="m-auto" :icon="coralIcons.addIcon" size="XL" alt="Larger" title="XL"></coral-icon>
                  <span class="m-auto"> Add/drop files </span>
                </div>
              </div>
            </div>
          </coral-fileupload>

        </div>

        <coral-fileupload v-if="!imageData" name="file" @change="selectIcon" class="m-auto" accept="image/png" multiple>
          <div coral-fileupload-dropzone=""  class="fileUpload-dropZone drop-zone"> 
            <div class="h-full" coral-fileupload-select="">
              <div class="drop-zone-wrapper">
                <coral-icon class="m-auto" :icon="coralIcons.addIcon" size="XL" alt="Larger" title="XL"></coral-icon>
                <span class="m-auto"> Add/drop files </span>
              </div>
            </div>
          </div>
        </coral-fileupload>

        <!-- <form class="coral-Form coral-Form--vertical" > -->
        <!-- </form> -->

        <section class="p-t-4">
          <div>
            <coral-checkbox id="isReupload">
              I'm re-uploading an icon that was previously on the site
            </coral-checkbox>
          </div>

          <div>
            <coral-checkbox id="isAuthor">I'm the original author of these icons. 
              <span class="opacity-80 p-l-4 f-w-200 coral-Body--XS">
                (It's ok if you aren't)
              </span>
            </coral-checkbox>
          </div>
        </section>

      </div>
  
    </coral-dialog-content>
    
    <coral-dialog-footer>
      <button is="coral-button" coral-close="">Cancel</button>
      <button v-if="imageData && email != '' " is="coral-button" variant="cta" @click="onUpload">Upload</button>
      <button v-if="!imageData || email == '' " is="coral-button" disabled>Upload</button>
    </coral-dialog-footer>

  </coral-dialog>
</template>

<script>
import Vue from 'vue'
import { mapActions } from 'vuex';
import Parse from 'parse/dist/parse.min.js';

import addIcon from "../assets/icons/add.svg"
import deleteIcon from "../assets/icons/delete.svg"
import newItemIcon from "../assets/icons/newItem.svg"

export default {
    name:"Dialog",
    props:{
    },
    data(){
      return{
        imageData: false,
        filesToShow: {},
        filesToUpload: {},
        coralIcons:{
          addIcon: addIcon,
          delete: deleteIcon,
          newItem: newItemIcon,
        },
        uploadProgress: 0,
        totalNumFiles: 0,
        email: "",
        credit: "",
        appName: "",
        yourName: "",
        isLoading: false
      }
    },
    methods:{
      ...mapActions(['showToast']),
      
      removeFile(e){
        let parent = this
        let iconName = e.target.id;
        Vue.delete(parent.filesToShow, iconName)
        Vue.delete(parent.filesToUpload, iconName)
        // If imageURL is empty, show the upload files component
        if (Object.keys(parent.filesToShow).length === 0) {
          parent.imageData = false
        }
      },

      selectIcon(event) {
        // Get selected image
        let parent = this
        let files = event.target.uploadQueue
        
        // Go through all the files that have been selected
        for(let fileNum in files){
          let file = files[fileNum].file
          let fileName = file.name.replace('.png', '')
          parent.filesToUpload[fileName] = file
          // Create URL of file to dislay back the image
          const objectURL = window.URL.createObjectURL(file);
          let value = {
            img: objectURL,
            name: fileName
          }
          parent.$set(parent.filesToShow, fileName, value)
        }
        parent.totalNumFiles = Object.keys(parent.filesToShow).length
        parent.imageData = true
      },

      async onUpload(){
        let parent = this
        
        // Get today's date
        var today = new Date();
        var dd = String(today.getDate()).padStart(2, '0');
        var mm = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
        var yyyy = today.getFullYear();

        today = dd + '/' + mm + '/' + yyyy;

        window.plausible("IconSubmission", {props: { date: today}})

        parent.isLoading = true
        let dialog = document.getElementById('submitIcon')
        let isReupload = document.getElementById('isReupload').checked
        let isAuthor = document.getElementById('isAuthor').checked

        for(let fileNum in parent.filesToUpload){
          let file =  parent.filesToUpload[fileNum];
          let appName = file.name.replace('.png', '');
          let fileName;
          
          if (/^[A-Za-z][A-Za-z0-9]*$/.test(file.name)) {
            fileName = `${file.name}`
          } else {
            let d = new Date()
            fileName = Math.round(Math.random()*10000 + d.getTime() )
            fileName = fileName.toString()
          }
          
          const Icons = Parse.Object.extend("Icons2");
          const icons = new Icons()

          const parseFile = new Parse.File(fileName, file); // Set file to new Parse object
          parseFile.save().then((uploaded) => {
            console.log("Success: ", uploaded._url);
            let iconUrl = uploaded._url.replace('http:', "https:")
            let currentUser = Parse.User.current()
          
            let dataToStore = {
              appName: appName,
              email: parent.email,
              credit: parent.credit,
              usersName: parent.yourName,
              uploadedBy: parent.yourName,
              fileName: fileName,
              highResPngFile: parseFile,
              highResPngUrl: iconUrl,
              isReupload: isReupload,
              isAuthor: isAuthor,
              timeStamp: Date.now(),
              approved: false,
              user: currentUser
            }

            icons.set(dataToStore);
            icons.save().then((icon) => { // Reset input boxes
              
              // Add icon relationship to user
              let userRelation = currentUser.relation("icons")
              userRelation.add(icons)
              currentUser.save()

              icon.set("alogliaID", icons.id);
              icon.save();

              parent.imageData = {},
                parent.picture= null,
                parent.uploadProgress++
                console.log("Document successfully written!");
                
                function clearInput(id){
                  document.getElementById(id).value = ""
                }
                
                Vue.delete(parent.filesToUpload, appName)
                Vue.delete(parent.filesToShow, appName)
               if (Object.keys(parent.filesToUpload).length === 0) {
                  parent.isLoading = false
                  parent.imageData = false
                  parent.email, parent.name = ""
                  parent.uploadProgress = 0
                  
                  let inputs = ["credit", "email-contributor", "yourName-contributor"]
                  for(let i in inputs){
                    clearInput(inputs[i])  
                  }

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
            console.log(error);
            parent.isLoading = false
            parent.showToast({
              id: "toastMessage",
              message: "There was an error, get in touch with @elrumo on Twitter",
              variant: "error"
            })
            // The file either could not be read, or could not be saved to Parse.
          });
        }
      },

      setEmail(e){
        console.log(e.target.value);
        this.email = e.target.value
      },

      saveCredit(e){
        console.log(e.target.value);
        this.credit = e.target.value
      },
      
      // setAppName(e){
      //   console.log(e.target.value);
      //   this.appName = e.target.value
      // },

      setYourName(e){
        console.log(e.target.value);
        this.yourName = e.target.value
      },
    },
}
</script>

<style>
</style>