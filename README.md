# SNS 회원가입/로그인 (+ FireBase Auth)

해당 기능 구현은 Firebase Auth 기능을 활용하여 구글 회원가입 / 로그인 , 카카오 회원가입 / 로그인 기능을 구현하였습니다.

그전에 Auth 기능 구현 방법부터 설명하겠습니다.

(해당 그림은 "생활코딩 auth"을 사용하였습니다. )

<img src = "https://user-images.githubusercontent.com/48902047/132115088-195e9068-cc52-4008-9442-574baa855208.png">

auth를 사용하는 순서에 대해 설명하겠습니다.

0. client는 Resource Sever 에 APi활용 신청을 통해 클라이언트 아이디와 시크릿을 받아온다.
1. Resource owner는 클라이언트에 접속한다.
2. Resource owner는 리소스 서버의 아이디와 비밀번호를 입력하고 인증요청을 받게 되며 승인을 한다.
3. 승인 요청과 함께 client는 Resource Server 에게 요청한다.
4. 요청된 resource owner의 정보가 맞는지 확인한다.
5. 맞다면, requestCode를 client에 넘겨준다.
6. client는 다시 클라이언트 아이디,시크릿,requestCode를 보내 확인을 다시 받는다.
7. Resource Sever는 세가지가 모두 맞다면 access token과 Resource owner의 유저 정보를 client에 넘겨준다.
8. client는 Resource owner의 access token키를 저장한다.
9. 해당 client는 같은 Resource owner 가 재 접속시 owner의 디바이스에 저장한 token키와 DB에 저장해 놓은 token이 일치하면 owner의 정보를 받아오며 로그인을 허가한다.

***
### :wrench: 프로젝트 사용 사례
<img src="https://user-images.githubusercontent.com/48902047/122346906-6f028180-cf84-11eb-801f-b3b6af8c6925.jpg" width="20%"></img>
<img src="https://user-images.githubusercontent.com/48902047/122347677-42029e80-cf85-11eb-8da6-7a44bd46dc83.png" width="20%"></img>
<img src="https://user-images.githubusercontent.com/48902047/132115389-521fd0ee-65db-4d8f-a9c7-a7b4a8108402.png" width="55%"></img>
+ [조치원 수호대](https://github.com/tnvnfdla1214/homemade_guardian) 
  + 구글 로그인/회원가입
  + 카카오 로그인 회원가입
+ [깃 허브 리파지토리](https://github.com/tnvnfdla1214/github_repository)
  + 깃 허브 로그인
***

### :lollipop: 설명 (조치원 수호대 구글 로그인 기능 설명)

#### 라이브러리와 Google Console API에 프로젝트를 등록

먼저 build.gradle 에 라이브러리를 추가합니다.

```Java
/*build.gradle*/

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])

    implementation 'androidx.appcompat:appcompat:1.1.0' //
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3' //
    implementation 'com.google.firebase:firebase-core:17.0.0'  //
    implementation 'com.google.firebase:firebase-analytics:17.4.4'
    
    // 카카오 로그인
    implementation group: 'com.kakao.sdk', name: 'usermgmt', version: '1.29.0' //

    //구글 로그인
    implementation 'com.google.android.gms:play-services-auth:18.0.0'//
}

```

#### LoginActivity 설계

먼저 해당 디바이스의 토큰의 정보가 FirebaseAuth에 토큰이 있는지 확인합니다.

있다면 자동로그인과 함께 토큰의 정보로 Firestore에서 정보를 찾아 해당 유저의 정보를 가져옵니다.

없다면 로그인 요청을 합니다.

승인이 됐다면 사용자의 기본 정보를 받아와 FireBaseAuth에 추가하며 회원가입을 유도합니다.

```Java
/*loginActivity.java*/

public class LoginActivity extends AppCompatActivity {
    private SessionCallback sessionCallback;

    private FirebaseAuth Firebaseauth =null;
    private FirebaseUser currentUser=null;

    private GoogleSignInClient mGoogleSignInClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        Firebaseauth = FirebaseAuth.getInstance();

        User_login_Check(); //유저 로그인 되어있는지 체크하는 함수
        FirebaseAuthgoogle(); //구글 로그인 메인 함수(onCreate안의 함수)

        //FirebaseAuthgoogle(); //구글 로그인 메인 함수(onCreate안의 함수)
    }

    //유저 로그인 되어있는지 체크하는 함수
    public void User_login_Check(){
        if (currentUser != null) {
            Intent intent = new Intent(getApplication(), MainActivity.class);
            startActivity(intent);
            finish();
        }
    }


    //구글 정보 기입 성공시 실행
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if(Session.getCurrentSession().handleActivityResult(requestCode, resultCode, data)) {
            super.onActivityResult(requestCode, resultCode, data);
            return;
        }
        if (requestCode == RC_SIGN_IN) {
            Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
            try {
                // Google Sign In was successful, authenticate with Firebase
                GoogleSignInAccount account = task.getResult(ApiException.class);
                firebaseAuthWithGoogle(account);
            } catch (ApiException e) {
            }
        }
    }


    //구글 로그인 메인 함수(onCreate안의 함수)
    public void FirebaseAuthgoogle(){
        // Configure Google Sign In
        GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build();
        mGoogleSignInClient = GoogleSignIn.getClient(this, gso);
        login_google.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                FirebaseAuthgoogle(); //구글 로그인 메인 함수(onCreate안의 함수)
                googlesignIn();
            }
        });
    }

    // 구글 회원가입
    private void googlesignIn() {
        Intent signInIntent = mGoogleSignInClient.getSignInIntent();
        startActivityForResult(signInIntent, RC_SIGN_IN);
    }

    //구글 로그인
    private void firebaseAuthWithGoogle(GoogleSignInAccount acct) {
        AuthCredential credential = GoogleAuthProvider.getCredential(acct.getIdToken(), null);
        Firebaseauth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            // Sign in success, update UI with the signed-in user's information
                            //추가 -가입할때 uid,id,userm,usermsg 만들기
                            currentUser = Firebaseauth.getCurrentUser();
                            updateUI(currentUser);
                        } else {
                            // 로그인이 실패하면 사용자에게 메시지를 표시하기
                            updateUI(null);
                            ChatUtil.showMessage(getApplicationContext(), task.getException().getMessage());
                        }
                    }
                });
    }
    
        private void updateUI(final FirebaseUser Curruntuser_uid) {
        if (Curruntuser_uid != null) {

            final String CurrentUid =Curruntuser_uid.getUid();
            FirebaseInstanceId.getInstance().getInstanceId().addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                @Override
                public void onComplete(@NonNull Task<InstanceIdResult> task) {
                    if (!task.isSuccessful()) {
                        return;
                    }

                   final String token = task.getResult().getToken();
                    final DocumentReference documentReference = FirebaseFirestore.getInstance().collection("USERS").document(CurrentUid);

                    documentReference.get().addOnSuccessListener(new OnSuccessListener<DocumentSnapshot>() {
                        @Override
                        public void onSuccess(DocumentSnapshot documentSnapshot) {
                            UserModel Usermodel = documentSnapshot.toObject(UserModel.class);
                            if (Usermodel != null) {
                                if (!token.equals(Usermodel.getUserModel_Token())) {
                                    Usermodel.setUserModel_Token(token);
                                    DocumentReference documentReferencesetUser = FirebaseFirestore.getInstance().collection("USERS").document(CurrentUid);
                                    documentReferencesetUser
                                            .update("UserModel_Token", token)
                                            .addOnSuccessListener(new OnSuccessListener<Void>() {
                                                @Override
                                                public void onSuccess(Void aVoid) {

                                                }
                                            })
                                            .addOnFailureListener(new OnFailureListener() {
                                                @Override
                                                public void onFailure(@NonNull Exception e) {

                                                }
                                            });
                                }
                            }
                        }
                    });

                }
            });
            Intent intent = new Intent(this, MainActivity.class);
            startActivity(intent);
            finish();
        }


}

```


