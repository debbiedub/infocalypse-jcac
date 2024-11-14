// The logic for every repo:
// 1. clone and throw the clone away unless a preserved copy is needed
// 2. pull to the preserved copy
// 3. refresh 2, 3, 5
//

// global variables
//   docker_image
//   docker_params

timestamps {
  stage('Prepare') {
    node ('debbies') {
      writeFile file:'Dockerfile', text: '''
FROM python:3.10

RUN pip3 install --upgrade pip
RUN pip3 install pyFreenet3
RUN pip3 install 'mercurial<6'
  ''';
      docker_params = "--network=host";
      docker_image = docker.build('hgfreenet:3');

      docker_image.inside(docker_params) {
	sh '''
	  if test -d dgof
	  then
	    (
	      export PATH=$PATH:$(pwd)/dgof
	      cd dgof
	      git pull --ff-only
	    ) || rm -rf dgof
	  fi
	  if test -d dgof
	  then
	    : dgof is updated
	  elif git clone http://localhost:8888/freenet:USK@nrDOd1piehaN7z7s~~IYwH-2eK7gcQ9wAtPMxD8xPEs,y61pkcoRy-ccB7BHvLCzt3RUjeMILf8ox26NKvPZ-jk,AQACAAE/dgof/26/ dgof 2> gitclone.err
	  then
	    cat gitclone.err 1>&2
	  else
	    cp gitclone.err newusk
	    sed -i '$s/.*USK@/USK@/p;d' newusk
	    sed -i 's,\\(/dgof/[0-9]*/\\).*,\\1,' newusk
	    git clone http://localhost:8888/freenet:$(cat newusk) dgof
	  fi
	  ''';

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
	    hg fn-setup --nofms --nowot
	  fi
	''';
      }
    }
  }
}

def gen_cl = { project, key ->
  boolean perm_done = false
  boolean toss_done = false
  boolean reinsert_done = false
  int reinsert_level = 2
  return {
    def perm_dir = "perm-$project"

    if (!perm_done) {
      def dir = perm_dir
      sh "export HOME=`pwd`; test -d ${dir} || hg clone freenet:${key} ${dir}"
      sh "export HOME=`pwd`; cd ${dir} && hg pull"
      sh "export HOME=`pwd`; cd ${dir} && hg log | egrep . > /dev/null"
      perm_done = true
      return 2000
    }

    if (!toss_done) {
      def dir = "throwaway-$project"
      sh script: "test -d ${dir} && rm -r ${dir}", returnStatus: true
      sh "export HOME=`pwd`; hg clone freenet:${key} ${dir}"
      sh "rm -r ${dir}"
      toss_done = true
      return 2000
    }

    if (!reinsert_done) {
      def level = reinsert_level++
      def dir = perm_dir
      sh "export HOME=`pwd`; cd ${dir} && hg fn-reinsert $level"
      if (level < 5) {
          return 2000
      }
      reinsert_done = true
    }

    return 0
  }
}

def buildParallelMap = [:]
def to_process = [
		  "infocalypse": "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/infocalypse.R1/31",
		  "fred": "USK@yJUguKTfUHgVutplApc8A3ucq~QogPfqx3-1ZunKjYk,EzZzXErTnhC~ll7HGpgDDik15KTFlwdpuGcRA7HL5uk,AQACAAE/fred.R1/0",
		  "pyFreenet": "USK@dqWzp0iGflRepXyBXHzxyKMSxq90kP2Lof8EdEr6woQ,nL53lCqG5ssdNtXMtVzTePJ4QYWGLkAhWMdmRqxwFjw,AQACAAE/pyFreenet.R1/1",
		  "hg-git": "USK@BTOaKIcNsNoa-z0qIAjXI0WAN25tNru8GtMCSsZh-sk,cZUVEEzoud2cdFHtrf4EC-EKsjvHBMRwvAzebqe4fjM,AQACAAE/hg-git.R1/1",
		  "b": "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/b.R1/1",
		  ]
timestamps {
  to_process.each { project, key ->
    buildParallelMap[project] = {
      stage(project) {
	def cl = gen_cl(project, key)
	def result = 1
	// The closure cl is run using cl() on the node
	// When it cannot do anymore (completed or needs to
	// sleep or wait for something) it returns.
	// If a positive value is returned, it sleeps this many seconds.
	// This allows others to use the node while this closure doesn't.
	while ({
	  node ('debbies') {
	    docker_image.inside(docker_params) {
	      result = cl()
		}
	  }
	  result > 0
	    }()) {
	  sleep(result)
	}
      }
    }
  }

  parallel(buildParallelMap)
}
