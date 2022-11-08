// The logic for every repo:
// 1. clone and throw the clone away unless a preserved copy is needed
// 2. pull to the preserved copy
// 3. refresh 2, 3, 5
//

def keys = [
  "infocalypse": "USK@key"
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
        if test -d infocalypse
	then
	  git pull --ff-only
	else
	  # Pull from the dgof mirror
	  git clone freenet::USK@Mm9MIkkeQhs~OMiCQ~83Vs48EvNwVRxjfeoFMOQHUYI,AxOZEuOyRM7oJjU43HFErhVw06ZIJLb8GMKNheWR3g4,AQACAAE/infocalypse/1/ infocalypse
	  echo "[extensions]" >> $HOME/.hgrc
	  echo "infocalypse=$(pwd)/infocalypse" >> $HOME/.hgrc
        fi
      '''
    }

    stage("clone") {
      def PROJECT = "infocalypse"
      sh "echo hg clone ${keys[$PROJECT]} throwaway-$PROJECT"
    }
    stage("pull") {
      def PROJECT = "infocalypse"
      sh "echo test -d perm-$PROJECT || hg clone ${keys[$PROJECT]} perm-$PROJECT"
      sh "echo cd perm-$PROJECT && hg pull"
    }
  }
}
