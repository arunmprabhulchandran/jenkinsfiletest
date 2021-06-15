def test() {
   sh '''
                    java -version
                '''
   
}

node{
    stage ("first"){
        echo "checking java version"
        test()
    }
}
