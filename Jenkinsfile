// The logic for every repo:
// 1. clone and throw the clone away unless a preserved copy is needed
// 2. pull to the preserved copy
// 3. refresh 2, 3, 5
//

// global variables
//   docker_image
//   docker_params

def saved_dir = "${env.SAVED_DIR}"

timestamps {
  stage('Prepare') {
    node ('debbies') {
      writeFile file:'Dockerfile', text: '''
FROM python:3.10

RUN pip3 install --upgrade pip
RUN pip3 install pyFreenet3
RUN pip3 install 'mercurial<6'
RUN pip3 install defusedxml pyyaml
RUN git clone http://localhost:8888/freenet:$(curl http://localhost:8888/freenet:USK@Mm9MIkkeQhs~OMiCQ~83Vs48EvNwVRxjfeoFMOQHUYI,AxOZEuOyRM7oJjU43HFErhVw06ZIJLb8GMKNheWR3g4,AQACAAE/infocalypse/0/ | sed '/Permanent/s/.*freenet://;s/".*//') /usr/local/src/infocalypse
  ''';
      docker_image = docker.build('hgfreenet:3', '--network=host .');
      docker_params = "--network=host -v $saved_dir:$saved_dir";
    }
  }
}

def gen_cl = { project, key ->
  boolean home_done = false
  boolean perm_done = false
  boolean toss_done = false
  boolean reinsert_done = false
  int toss_laps = 1
  int reinsert_level = 2
  return {
    def perm_dir = "$saved_dir/perm-$project"
    def home_dir = "$saved_dir/home-$project"

    if (!home_done) {
      sh "HOME=$home_dir; " + '''
if test -d $HOME
then
  : 
else
  mkdir $HOME
  export HOME
  echo "[extensions]" >> $HOME/.hgrc
  echo "infocalypse=/usr/local/src/infocalypse/infocalypse" >> $HOME/.hgrc
  hg fn-setup --nofms --nowot
fi
'''
      home_done = true
    }

    if (!perm_done) {
      def dir = perm_dir
      sh "test -d ${dir} || HOME=$home_dir hg clone freenet:${key} ${dir}"
      sh "cd ${dir} && HOME=$home_dir hg pull"
      sh "cd ${dir} && HOME=$home_dir hg log | egrep . > /dev/null"
      perm_done = true
      return 2000
    }

    if (!toss_done) {
      def lap = toss_laps++
      def dir = "/tmp/throwaway-$project"
      sh script: "test -d ${dir} && rm -r ${dir}", returnStatus: true
      sh "HOME=$home_dir hg clone freenet:${key} ${dir}"
      def result = sh script: "cd ${dir} && HOME=$home_dir hg log | egrep . > /dev/null", returnStatus: true
      if (result != 0) {
        if (lap < 3) {
	  echo "$project: Could not clone the repo lap $lap - will retry"
	  return 1800 + lap * 100
	}
        unstable "$project: Could not clone the repo. Giving up after $lap attempts."
      }
      sh "rm -r ${dir}"
      toss_done = true
      return 2000
    }

    if (!reinsert_done) {
      if (reinsert_level == 4) {
        // Skip level 4 of reinsert
        reinsert_level++
      }
      def level = reinsert_level++
      def dir = perm_dir
      sh "cd ${dir} && HOME=$home_dir hg fn-reinsert --level $level"
      if (reinsert_level < 5) {
        return 2000
      }
      reinsert_done = true
    }

    return 0
  }
}

def buildParallelMap = [:]
def to_process = [
		  "infocalypse":                   "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/infocalypse.R1/31",
		  "fred":                          "USK@yJUguKTfUHgVutplApc8A3ucq~QogPfqx3-1ZunKjYk,EzZzXErTnhC~ll7HGpgDDik15KTFlwdpuGcRA7HL5uk,AQACAAE/fred.R1/0",
		  "pyFreenet1":                     "USK@-sEkUkv7SClGNhjpvFF9TtmGHnoiwvThkepMUTy7Fh4,Rciq6x0804hb~rNzxmINfkuSCgAEvHZjJNHQcUo0nuQ,AQACAAE/pyfreenet.R1/0",
		  "pyFreenet":                     "USK@dqWzp0iGflRepXyBXHzxyKMSxq90kP2Lof8EdEr6woQ,nL53lCqG5ssdNtXMtVzTePJ4QYWGLkAhWMdmRqxwFjw,AQACAAE/pyFreenet.R1/1",
		  // "hg-git":                        "USK@hRSO9yNEENHalLqm8KCjn0XCSk-DlLnALp4CaXunjLs,TNRUMEkbKtIPNJPf-HV9ykWl4cEcmIob3wFluZZzYGk,AQACAAE/hg-git.R1/8",
		  // "hg-git":                        "USK@BTOaKIcNsNoa-z0qIAjXI0WAN25tNru8GtMCSsZh-sk,cZUVEEzoud2cdFHtrf4EC-EKsjvHBMRwvAzebqe4fjM,AQACAAE/hg-git.R1/1",
		  "b":                             "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/b.R1/1",
		  // "infocalypse-dvcs-over-freenet": "USK@ugGhE7lfI-QyFYL6yuZt7uyhlrVfvIRo0xkQxd2qV~c,L1bCwDm5Hh5qgqml7SBQcF85vL8IPvzN4WihjWIKH8I,AQACAAE/infocalypse-dvcs-over-freenet.R1/15",
		  // "wiki_hacking":                  "USK@kRM~jJVREwnN2qnA8R0Vt8HmpfRzBZ0j4rHC2cQ-0hw,2xcoQVdQLyqfTpF2DpkdUIbHFCeL4W~2X1phUYymnhM,AQACAAE/wiki_hacking.R1/24",
		  // "infocalypse-repo":              "USK@j2Mfv0b81MNSslh3Qg9gs1vgQj36whsuosP8ydZq158,Xb~vh8XEXmdwGFca4HAx-ILOZD6AWPVuRzIuLerl4PU,AQACAAE/infocalypse-repo.R1/20",
		  // "fred1":                         "USK@3Xvf~RI-N8YC-xFtQIEnEwR~ZLC05UhQjQzKAS0lL4Y,PqTg9Y5Me6~WthiW5FoHG9Mq18c7oRbe0owRutJJJZc,AQACAAE/fred1.R1/124",
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
