// The logic for every repo:
// 1. clone and throw the clone away unless a preserved copy is needed
// 2. pull to the preserved copy
// 3. refresh 2, 3, 5
//

def keys = [
  "infocalypse": "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/infocalypse.R1/29"
]

node ('debbies') {
  writeFile file:'Dockerfile', text: '''
FROM python:3

RUN pip3 install pyFreenet3
RUN pip3 install mercurial
  '''
  docker.build('hgfreenet:3').inside("--network=host") {

    stage('Get infocalypse') {
      sh '''
        if test -d dgof
	then
          (
            export PATH=$PATH:$(pwd)/dgof
            cd dgof
	    git pull --ff-only
	  )
        elif git clone http://localhost:8888/freenet:USK@nrDOd1piehaN7z7s~~IYwH-2eK7gcQ9wAtPMxD8xPEs,y61pkcoRy-ccB7BHvLCzt3RUjeMILf8ox26NKvPZ-jk,AQACAAE/dgof/26/ dgof 2> gitclone.out
        then
          cat gitclone.out 1>&2
        else
          cp gitclone.out newusk
          sed -i '$s/.*USK@/USK@/p;d' newusk
          sed -i 's,\\(/dgof/[0-9]*/\\).*,\\1,' newusk
          git clone http://localhost:8888/freenet:$(cat newusk) dgof
        fi
        '''

      sh '''
        export PATH=$PATH:$(pwd)/dgof
	export HOME=`pwd`
        if test -d infocalypse
	then
	  (
	    cd infocalypse
            git pull --ff-only
	  )
	else
	  # Pull from the dgof mirror
	  git clone freenet::USK@Mm9MIkkeQhs~OMiCQ~83Vs48EvNwVRxjfeoFMOQHUYI,AxOZEuOyRM7oJjU43HFErhVw06ZIJLb8GMKNheWR3g4,AQACAAE/infocalypse/1/ infocalypse
	  echo "[extensions]" >> $HOME/.hgrc
	  echo "infocalypse=$(pwd)/infocalypse/infocalypse" >> $HOME/.hgrc
	  hg fn-setup --nofms --nowot1
        fi
      '''
    }

    def clone(project, key) {
      stage("clone-${project}" ) {
        def dir = "throwaway-$project"
        sh script: "test -d ${dir} && rm -r ${dir}", returnStatus: true
        sh "export HOME=`pwd`; hg clone freenet:${key} ${dir}"
        sh "rm -r ${dir}"
      }
    }

    def pull(project, key) {
      stage("pull-${project}") {
        def dir = "perm-$project"
        sh "export HOME=`pwd`; test -d ${dir} || hg clone freenet:${key} ${dir}"
        sh "export HOME=`pwd`; cd ${dir} && hg pull"
      }
    }

    def project = "infocalypse"
    def key = keys[PROJECT]
    clone(project, key);
    pull(project, key);
  }
}
