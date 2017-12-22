> This is a note about how I did build RPM via Jenkins job previously
> It was tested on a CentOS 7 virtual machine

## Jenkins Job Settings


### Creating a Jenkins Job
| Category	| Name	| Value	| 
| --- | --- | --- | --- |
| Project Type	| Pipeline	| |
| General	| Project name	| Building-bash-RPM	|
|| Description	| Generate bash RPM package project |
| |Discard old builds | Strategy: Log Rotate<br/>Days to keep builds: 31<br/>Max # of builds to keep: 7|
| Build Triggers	| | Gerrit event|
| Gerrit Trigger	| Choose a Server	| Gerrit |
| | Trigger on	| Patchset Created|
| | | Gerrit Project: (Plain) rpmbuild-el7-bash<br/>Gerrit Branches: (Plain) master |
|Pipeline	|Definition	| Pipeline script|
| | Script | *See below* |


### Script	content

```groovy
node('rpmbuild'){
    stage('Preparation'){
        git 'https://gerrit.example.local/rpmbuild-el7-bash'
        try{
            // Fetch the changeset to a local branch using the build parameters provided to the
            // build by the Gerrit plugin...
            def changeBranch = "change-${GERRIT_CHANGE_NUMBER}-${GERRIT_PATCHSET_NUMBER}"    
            sh "git fetch origin ${GERRIT_REFSPEC}:${changeBranch}"
            sh "git checkout ${changeBranch}"
        }catch(MissingPropertyException e){
            // GERRIT_CHANGE_NUMBER, GERRIT_PATCHSET_NUMBER.. can not be found if any user run Build Now
        }
    }
    stage('Build'){
        sh '/bin/rpmbuild --define "release $(date +%Y%m%d%H%M%S)" --define "_topdir ${WORKSPACE}" -ba ${WORKSPACE}/SPECS/bash.spec'
    }
    stage('Results'){
        archiveArtifacts artifacts: 'RPMS/x86_64/*.rpm'
        archiveArtifacts artifacts: 'SRPMS/*.rpm'
        sh "/bin/rm -rf RPMS/x86_64/*.rpm SRPMS/*.rpm"
    }
}
```

### Build & Check
TODO



