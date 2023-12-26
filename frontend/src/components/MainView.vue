<template>
  <div class="mt-16 grid grid-cols-2">
    <div>
      <div v-if="signInStep === 'SIGN_UP'" class="flex flex-col w-80 gap-y-2">
        <h1 class="font-semibold text-4xl">Sign Up</h1>
        <input class="border-2 p-2 rounded-sm" type="text" v-model="email" placeholder="Email" />
        <input class="border-2 p-2 rounded-sm" type="password" v-model="password" placeholder="Password" />
        <button class="mt-2 py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="signUp">Sign Up</button>
      </div>
  
      <div v-if="signInStep === 'VERIFY'" class="flex flex-col w-80 gap-y-2">
        <h1 class="font-semibold text-4xl">Sign Up</h1>
        <input class="border-2 p-2 rounded-sm" type="text" v-model="verificationCode" placeholder="Verification Code" />
        <button class="mt-2 py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="verify">Confirm Sign Up</button>
      </div>
  
      <div v-if="signInStep === 'SIGN_IN'" class="flex flex-col w-80 gap-y-2">
        <h1 class="font-semibold text-4xl">Sign In</h1>
        <input class="border-2 p-2 rounded-sm" type="text" v-model="email" placeholder="Email" />
        <input class="border-2 p-2 rounded-sm" type="password" v-model="password" placeholder="Password" />
        <button class="mt-2 py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="signIn">Sign In</button>
        <button class="underline text-left" @click="signInStep = 'SIGN_UP'">Register</button>
      </div>
  
      <div v-if="signInStep === 'SIGNED_IN'" class="flex flex-col w-80 gap-y-2">
        <h1 class="font-semibold text-4xl">You're signed in</h1>
        <button class="underline text-left" @click="signOut">Sign Out</button>
      </div>
    </div>
    <div v-if="cred !== null && signInStep === 'SIGNED_IN'" class="flex flex-col gap-y-10">
      <div class="flex flex-col gap-y-2">
        <h1 class="font-semibold text-4xl">DynamoDB</h1>
        <div class="flex flex-col max-w-60 gap-y-2">
          <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            @click="putDDBAuthed">Put (authorized)</button>
          <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            @click="getDDBAuthed">Get (authorized)</button>
          <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            @click="putDDBUnauthed">Put (unauthorized)</button>
          <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            @click="getDDBUnauthed">Get (unauthorized)</button>
        </div>
      </div>
      
      <div class="flex flex-col max-w-60 gap-y-2">
        <h1 class="font-semibold text-4xl">S3</h1>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="putS3Authed">Put (authorized)</button>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="getS3Authed">Get (authorized)</button>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="putS3UnauthedUser">Put (unauthorized user)</button>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="getS3UnauthedUser">Get (unauthorized user)</button>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="putS3UnauthedObject">Put (unauthorized object)</button>
        <button class="py-2 px-4 border border-transparent font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          @click="getS3UnauthedObject">Get (unauthorized object)</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Amplify } from 'aws-amplify'
import * as Auth from 'aws-amplify/auth'
import { 
  CognitoIdentityClient, 
  GetIdCommand, 
  GetCredentialsForIdentityCommand
} from '@aws-sdk/client-cognito-identity'
import { DynamoDBClient } from '@aws-sdk/client-dynamodb'
import { DynamoDBDocument } from '@aws-sdk/lib-dynamodb'
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3'

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      userPoolClientId: import.meta.env.VITE_COGNITO_USER_POOL_CLIENT_ID
    }
  }
})

const cognitoClient = new CognitoIdentityClient({ region: import.meta.env.VITE_AWS_REGION })
let documentClient
let s3Client

const email = ref('')
const password = ref('')
const verificationCode = ref('')
const signInStep = ref('SIGN_IN')
const user = ref(null)
const cred = ref(null)
const identityId = ref(null)

onMounted(async () => {
  try {
    const session = await Auth.fetchAuthSession()
    if (session.tokens) {
      signInStep.value = 'SIGNED_IN'
      user.value = await Auth.getCurrentUser()
      console.log(user)

      cred.value = await getCredentials(session)
    }
  } catch (error) {
    console.log(error)
  }
})

async function signUp() {
  try {
    const signUpResult = await Auth.signUp({
      username: email.value,
      password: password.value
    })
    console.log(signUpResult.user)
    signInStep.value = 'VERIFY'
  } catch (error) {
    alert(error.message)
  }
}

async function verify() {
  try {
    const confirmSignUpResult = await Auth.confirmSignUp({
      username: email.value,
      confirmationCode: verificationCode.value
    })
    console.log(confirmSignUpResult)
    signInStep.value = 'SIGN_IN'
  } catch (error) {
    alert(error.message)
  }
}

async function signIn() {
  try {
    const signInResult = await Auth.signIn({
      username: email.value,
      password: password.value
    })
    console.log(signInResult)
    
    user.value = await Auth.getCurrentUser()
    console.log(user)

    const session = await Auth.fetchAuthSession()
    cred.value = await getCredentials(session)

    signInStep.value = 'SIGNED_IN'

    alert('you are signed in')
  } catch (error) {
    alert(error.message)
  }
}

async function getCredentials(session) {
  const getIdResp = await cognitoClient.send(new GetIdCommand({
    IdentityPoolId: import.meta.env.VITE_COGNITO_IDENTITY_POOL_ID,
    Logins: {
      [import.meta.env.VITE_COGNITO_PROVIDER_NAME]: session.tokens.idToken.toString()
    }
  }))
  identityId.value = getIdResp.IdentityId

  const getCredResp = await cognitoClient.send(new GetCredentialsForIdentityCommand({
    IdentityId: identityId.value,
    Logins: {
      [import.meta.env.VITE_COGNITO_PROVIDER_NAME]: session.tokens.idToken.toString()
    }
  }))
  console.log(getCredResp)

  const dynamoDBClient = new DynamoDBClient({ 
    region: import.meta.env.VITE_AWS_REGION,
    credentials: {
      accessKeyId: getCredResp.Credentials.AccessKeyId,
      secretAccessKey: getCredResp.Credentials.SecretKey,
      sessionToken: getCredResp.Credentials.SessionToken
    }
  })
  documentClient = DynamoDBDocument.from(dynamoDBClient)
  s3Client = new S3Client({
    region: import.meta.env.VITE_AWS_REGION,
    credentials: {
      accessKeyId: getCredResp.Credentials.AccessKeyId,
      secretAccessKey: getCredResp.Credentials.SecretKey,
      sessionToken: getCredResp.Credentials.SessionToken
    }
  })

  return getCredResp.Credentials
}

async function signOut() {
  try {
    await Auth.signOut()
    user.value = null
    email.value = ''
    password.value = ''

    signInStep.value = 'SIGN_IN'

    alert('you are signed out')
  } catch (error) {
    alert(error.message)
  }
}

async function putDDBAuthed() {
  await documentClient.put({
    TableName: import.meta.env.VITE_DYNAMODB_TABLE_NAME,
    Item: {
      userId: identityId.value,
      timestamp: new Date().toISOString()
    }
  })

  alert('put item succeded')
}

async function putDDBUnauthed() {
  try {
    await documentClient.put({
      TableName: import.meta.env.VITE_DYNAMODB_TABLE_NAME,
      Item: {
        userId: 'some-other-user',
        timestamp: new Date().toISOString()
      }
    })
  
    alert("put item succeded but it shouldn't have!!")
  } catch(err) {
    alert('put item failed:' + err.message)  
  }
}

async function getDDBAuthed() {
  const resp = await documentClient.get({
    TableName: import.meta.env.VITE_DYNAMODB_TABLE_NAME,
    Key: {
      userId: identityId.value
    }
  })

  alert('get item succeded:\n' + JSON.stringify(resp.Item, null, 2))
}

async function getDDBUnauthed() {
  try {
    await documentClient.get({
      TableName: import.meta.env.VITE_DYNAMODB_TABLE_NAME,
      Key: {
        userId: user.value.userId
      }
    })
  
    alert("get item succeded but it shouldn't have!!")
  } catch (err) {
    alert('get item failed:' + err.message)  
  }
}

async function putS3Authed() {
  await s3Client.send(new PutObjectCommand({
    Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
    Key: `${identityId.value}/test.json`,
    Body: JSON.stringify({
      userId: identityId.value,
      timestamp: new Date().toISOString()
    })
  }))

  alert('put item succeded')
}

async function putS3UnauthedUser() {
  try {
    await s3Client.send(new PutObjectCommand({
      Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
      Key: 'some-other-user/test.json',
      Body: JSON.stringify({
        userId: identityId.value,
        timestamp: new Date().toISOString()
      })
    }))
  
    alert("put item succeded but it shouldn't have!!")
  } catch(err) {
    alert('put item failed:' + err.message)  
  }
}

async function putS3UnauthedObject() {
  try {
    await s3Client.send(new PutObjectCommand({
      Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
      Key: `${identityId.value}/some-other-file.json`,
      Body: JSON.stringify({
        userId: identityId.value,
        timestamp: new Date().toISOString()
      })
    }))
  
    alert("put item succeded but it shouldn't have!!")
  } catch(err) {
    alert('put item failed:' + err.message)  
  }
}

async function getS3Authed() {
  const resp = await s3Client.send(new GetObjectCommand({
    Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
    Key: `${identityId.value}/test.json`
  }))

  const body = await streamToString(resp.Body)

  alert('get item succeded:\n' + body)
}

async function getS3UnauthedUser() {
  try {
    await s3Client.send(new GetObjectCommand({
      Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
      Key: 'some-other-user/test.json'
    }))
  
    alert("get item succeded but it shouldn't have!!")
  } catch (err) {
    alert('get item failed:' + err.message)  
  }
}

async function getS3UnauthedObject() {
  try {
    await s3Client.send(new GetObjectCommand({
      Bucket: import.meta.env.VITE_S3_BUCKET_NAME,
      Key: `${identityId.value}/some-other-file.json`
    }))
  
    alert("get item succeded but it shouldn't have!!")
  } catch (err) {
    alert('get item failed:' + err.message)  
  }
}

async function streamToString(stream) {
  return new Promise((resolve, reject) => {
    const reader = stream.getReader()
    let chunks = ''

    reader.read().then(function processText({ done, value }) {
      if (done) {
        resolve(chunks)
        return
      }

      chunks += new TextDecoder().decode(value, { stream: true })
      return reader.read().then(processText)
    }).catch(reject)
  })
}

</script>