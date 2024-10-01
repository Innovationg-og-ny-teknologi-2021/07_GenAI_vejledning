# 07_GenAI_ChatBot_vejledning 🤖

I dag skal vi arbejde med at implementere en GPT-4o mini chatbot i en react-native applikation.

## Opsætning af projekt

Start med at oprette et nyt react-native projekt med expo-cli. 

``` npx create-expo-app GenAI_Chat --template blank```

Navigere til projektmappen med cd og kør projektet med ```npm start```

Når projektet er startet op, kan du åbne det i en simulator eller på din telefon. 
Sluk herefter serveren og installer de nødvendige komponenter med:

Estat jeres package.json dependencies med følgende:
```javascript
  "dependencies": {
    "@react-native-async-storage/async-storage": "^1.19.3",
    "@react-navigation/stack": "^6.4.1",
    "expo": "~51.0.28",
    "expo-status-bar": "~1.12.1",
    "openai": "^4.65.0",
    "react": "18.2.0",
    "react-native": "0.74.5",
    "react-native-async-storage": "^0.0.1",
    "react-native-gifted-chat": "^2.6.3",
    "react-native-reanimated": "~3.10.1"
  },
``` 

Kør herefter `npm install` i jeres project så I downloader de nødvendige pakker.

Som I kan se fra overstående dependencies, skal vi arbejde med OpenAI og deres api. 

## Stack navigation

Vi skal bruge en stack navigation til at navigere mellem vores sider.

Opret en ny mappe kaldt components og i den mappe opret en fil ved navn `StackNavigator.js`

I StackNavigator.js skal vi importere de nødvendige komponenter og oprette en stack navigation med `const Stack=createStackNavigator();`

Stack navigationen skal have 2 sider, en til vores chat og en til vores home screen.
Opret en mappe mere med navn `screens` med følgende filer i:
- ChatScreen.js
- HomeScreen.js

Impotere disse screens i din `StackNavigator.js` og indsæt dm i din navigation:
```javascript
export default function HomeNavigation() {
  return (
    <NavigationContainer>
    <Stack.Navigator screenOptions={{}}>
        <Stack.Screen name="home" component={HomeScreen} 
        options={{headerShown:false}} />
        <Stack.Screen name="chat" component={ChatScreen}  />
    </Stack.Navigator>
    </NavigationContainer>
  )
}
```

### App.js

Vi skal nu tilbage til vores App.js og importere vores StackNavigator. <br>

```javascript
export default function App() {
  return (
    <HomeNavigation />
  );
}
```

I skal også slette jeres styles i App.js og huske at importer de nødvendige komponenter

## HomeScreen.js

Før vi kan arbejde med vores chatbot, skal vi have lavet en home screen hvor vi kan vælge at starte en chat.

Start med at importere de nødvendige komponenter:
```javascript
import {
    View,
    Text,
    Image,
    FlatList,
    TouchableOpacity,
    Dimensions,
  } from "react-native";
//VIGTIG!
import React, { useEffect, useState } from "react";
import AsyncStorage from "@react-native-async-storage/async-storage";

import ChatFaceData from "../services/ChatFaceData";
```

### HomeScreen logik
I vores **HomeScreen function** skal vi oprette 2 useStates til at hente vores valgte chatbots.
Vi skal også hente vores navigation. 

```javascript
export default function HomeScreen({navigation}) {
    //Håndtere ChatBotValg
    const [chatFaceData, setChatFaceData] = useState([]);
    const [selectedChatFace, setSelectedChatFace] = useState([]);
}
```

Dette er en React Hook ved navn useState, der bruges til at oprette en tilstandsvariabel og en funktion til at opdatere denne tilstand.

``chatFaceData`` er en tilstandsvariabel, som initialt er sat til et tomt array.

``setChatFaceData`` er en funktion, der kan bruges til at opdatere værdien af chatFaceData.

Det samme gælder for ``selectedChatFace``.

Herefter laver vi en useEffect til at hente vores valgte chatbot.
```javascript
//Håndtere valg af chatbot
  useEffect(() => {
    setChatFaceData(ChatFaceData);
    checkFaceId();
  }, []);
```

#### checkFaceId function
Vi skal også lave en asynkron funktion `` checkFaceId `` der gemmer vores valg af chatbot i vores AsyncStorage.
```javascript
//Håndere valg af chatbot (Gemmer valg)
  const checkFaceId = async () => {
    const id = await AsyncStorage.getItem("chatFaceId");
    id
      ? setSelectedChatFace(ChatFaceData[id])
      : setSelectedChatFace(ChatFaceData[0]);
  };
```

#### onChatFacePress function
Til sidst laver vi en funktion der håndtere at vi kan skifte imellem dem.
```javascript
//Håndtere valg af chatbot (Skifte mellem dem)
  const onChatFacePress = async (id) => {
    setSelectedChatFace(ChatFaceData[id - 1]);
    await AsyncStorage.setItem("chatFaceId", (id - 1).toString());
  };
```

### Return function i HomeScreen
Vi skal lave 2 views til at wrappe resten af vores frontend. Vi laver også inline styling da dette er en ret lille app. 

```javascript
return (
    //Vi laver de to views til at rappe resten af frontend
    <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
      <View style={{ alignItems: "center" }}>
        
      </View>
    </View>
  );
```

Lad os starte med at indsætte følgende kode i dit view.
```javascript
{/*Titel*/}
<Text style={[{ color: selectedChatFace?.primary }, { fontSize: 30 }]}>
    Hello,
</Text>
{/*Undertitel - Bot Intro*/}
<Text
    style={[
    { color: selectedChatFace?.primary },
    { fontSize: 30, fontWeight: "bold" },
    ]}
>
    I am {selectedChatFace.name}
</Text>
{/*Bot Image*/}
<Image
    source={{ uri: selectedChatFace.image }}
    style={{ height: 150, width: 150, marginTop: 20 }}
/>
{/*Besked*/}
<Text style={{ marginTop: 30, fontSize: 25 }}>How Can I help you?</Text>
```

Nu skal vi lave endnu et view med en flatliste som indeholder vores chatbots.
```javascript
{/*Valg af bots*/}
<View
    style={{
    marginTop: 20,
    backgroundColor: "#F5F5F5",
    alignItems: "center",
    height: 110,
    padding: 10,
    borderRadius: 10,
    }}
>
    {/*Liste med de forskellige bots*/}
    <FlatList
    data={chatFaceData}
    horizontal={true}
    renderItem={({ item }) =>
        item.id != selectedChatFace.id && (
        <TouchableOpacity
            style={{ margin: 15 }}
            onPress={() => onChatFacePress(item.id)}
        >
            <Image
            source={{ uri: item.image }}
            style={{ width: 40, height: 40 }}
            />
        </TouchableOpacity>
        )
    }
    />
    <Text style={{ marginTop: 5, fontSize: 17, color: "#B0B0B0" }}>
    Choose Your Fav ChatBuddy
    </Text>
</View>
```

Til sidst skal vi lave en knap til at starte chatten.

```javascript
{/*Knap til chatten*/}
<TouchableOpacity
    style={[
    { backgroundColor: selectedChatFace.primary },
    {
        marginTop: 40,
        padding: 17,
        width: Dimensions.get("screen").width * 0.6,
        borderRadius: 100,
        alignItems: "center",
    },
    ]}
    onPress={() => navgitaion.navigate("chat")}
>
    <Text style={{ fontSize: 16, color: "#fff" }}>Let's Chat</Text>
</TouchableOpacity>
```

Din HomeScreen er nu næsten done.

### Hent vores ChatBots

Vi skal nu hente vores ChatBots fra en ny mappe.

Lav en ny mappe og kald den `services`.

I services mappen skal du oprette en fil kaldt `ChatFaceData.js`.

I ChatFaceData.js skal du indsætte følgende kode:
```javascript
//ChatBot Data
export default chatFaceData=[
    {
        id:1,
        name:'Noyi',
        image:'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685678135/chat_1_c7eda483e3.png',
        primary:'#FFC107',
        secondary:''
    },
    {
        id:2,
        name:'Pogu',
        image:'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_21_2e18bb4a61.png',
        primary:'#E53057',
        secondary:''
    },
    {
        id:3,
        name:'Nista',
        image:'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_22_409561b953.png',
        primary:'#3B96D2',
        secondary:''
    },
    {
        id:4,
        name:'Estor',
        image:'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_18_893d24cebc.png',
        primary:'#37474F',
        secondary:''
    },
    {
        id:5,
        name:'Pega',
        image:'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_23_211d7370cb.png',
        primary:'#2473FE',
        secondary:''
    },
]
```

Nu skal du importer ChatFaceData i din HomeScreen.

```javascript
import ChatFaceData from "../services/ChatFaceData";
```

<b> Du er nu done med HomeScreen! ┗(＾0＾)┓ </b>

## ChatScreen og API

### Request.js

Start med at oprette en ny fil i din services mappe kaldt `Request.js`.

I `Request.js` skal du indsætte følgende kode (dog med din egen API nøgle):
```javascript
import OpenAI from "openai/index.mjs";

// Create a new instance of the OpenAI class
const openai = new OpenAI({apiKey: "YOUR_API_KEY"});

// Create a function that sends a message to the OpenAI API
export default async function SendMessage(messageArray) {
    const response = await openai.chat.completions.create({
        model: "gpt-4o-mini",
        messages: messageArray,
    });

    // Extract the AI's reply from the response
    const result = response.choices[0]?.message?.content || "";
    // Return the AI's reply
    return { role: "assistant", content: result };
}
```

### ChatScreen.js
Vi skal nu oprette en ny fil i vores screens mappe kaldt `ChatScreen.js.`

I `ChatScreen.js` skal du importer følgende:
```javascript
import { View } from 'react-native';
import React from 'react';
import { SafeAreaView } from 'react-native';

import { Bubble, GiftedChat, InputToolbar, Send } from 'react-native-gifted-chat';

import {useState,useEffect, useCallback} from 'react';
import { FontAwesome } from '@expo/vector-icons';

import AsyncStorage from '@react-native-async-storage/async-storage';

import ChatFaceData from '../services/ChatFaceData';
import SendMessage from '../services/Request';

CHAT_BOT_FACE='https://res.cloudinary.com/dknvsbuyy/image/upload/v1685678135/chat_1_c7eda483e3.png'
```

Vi skal nu lave 3 useStates til at håndtere vores chat. <br>

```javascript
export default function ChatScreen() {

    const [messages, setMessages] = useState([]);
    const [loading, setLoading] = useState(false);
    const [chatFaceColor,setChatFaceColor]=useState();
    
```

og en useEffect til at hente vores valgte chatbot. <br>

```javascript
//Håndtere valgt af ChatBot
    useEffect(() => {
        checkFaceId();
    }, [])
```

### Chatbottens Function Definition

For at definere vores chatbots function, definere vi den således: 

```javascript
//Definere vores chat "hukkomelse"
    const conversationHistory = [ {role: 'system', content: 'You are an assistant that serves as a tutor for master’s students learning business model theory. You adapt to the student’s knowledge level and guide them to discover answers on their own, avoiding direct solutions. You have a strong understanding of Chesbrough’s core ideas and ensure conversations stay focused on business model theory. Encourage critical thinking and use questions to deepen the student’s understanding. You are not supposed to answer any other questions that are not related to business model theory.'},
        { role: 'assistant', content: 'Hello, I am your assistant. How can I help you?' }
    ];
```
Indsæt dette efter din `useEffect`

#### checkFaceId function
Nu skal vi lave en function der håndtere vores valgte chatbot og sender en valgt besked. <br>

```javascript
//Sætter den valgte chatbot til og sender den første besked
    const checkFaceId=async()=>{
        const id= await AsyncStorage.getItem('chatFaceId');
       CHAT_BOT_FACE= id?ChatFaceData[id].image: ChatFaceData[0].image;
       setChatFaceColor(ChatFaceData[id].primary);
       setMessages([
        {
          _id: 1,
          text: 'Hello, I am '+ChatFaceData[id].name+', How Can I help you?',
          createdAt: new Date(),
          user: {
            _id: 2,
            name: 'React Native',
            avatar: CHAT_BOT_FACE,
            },
        },
      ])
    }
```
Kort fortalt: checkFaceId funktionen tjekker, hvilken chatbot der tidligere er blevet valgt (hvis nogen), sætter chatbotens visuelle repræsentation, og initialiserer en introduktionsbesked fra den valgte chatbot.


#### onSend function
Vi skal nu lave en function der håndtere når vi sender en besked. <br>

```javascript
//Håndtere Chatten
    const onSend = useCallback((messages = []) => {
       
      setMessages(previousMessages => GiftedChat.append(previousMessages, messages))
      if(messages[0].text)
      {
        getBardResp(messages[0].text);
      }
    }, [])
```


#### getBardResp function
Vi skal nu lave en function der håndtere når vi modtager en besked fra vores bot <br>

```javascript
const getBardResp = (msg) => {
        setLoading(true)

        //Sætter brugerens besked i vores chat "hukkomelse"
        const userMessage = { role: 'user', content: msg };
        conversationHistory.push(userMessage);
        
        //SendMessage er vores funktion fra RequestPage.js
        SendMessage(conversationHistory)
        .then(response => {
            if (response.content) {
                setLoading(false)

                //Sætter chat AI's svar i den format gifted chat forventer.
                const chatAIResp = {
                    _id: Math.random() * (9999999 - 1),
                    text: response.content,
                    createdAt: new Date(),
                    user: {
                      _id: 2,
                      name: 'React Native',
                      avatar: CHAT_BOT_FACE,
                    }
                }
                //Sætter chat AI's svar i vores chat "hukkomelse"
                const botMessage = { role: 'assistant', content: response.content };
                conversationHistory.push(botMessage);
                
                //Sætter chat AI's svar i vores chat
                setMessages(previousMessages => GiftedChat.append(previousMessages, chatAIResp))  
            } else {
                //Hvis der ikke er et svar fra AI
                setLoading(false)
                
                const chatAIResp = {
                    _id: Math.random() * (9999999 - 1),
                    text: "Sorry, I cannot help with it",
                    createdAt: new Date(),
                    user: {
                      _id: 2,
                      name: 'React Native',
                      avatar: CHAT_BOT_FACE,
                    }
                }
    
                setMessages(previousMessages => GiftedChat.append(previousMessages, chatAIResp))  
            }
        })
        .catch(error => {
            console.error(error);
            // Handle error further if needed
        });
    }
```

Funktionen getBardResp tager en parameter msg. Når den kaldes, sætter den tilstanden loading til true, hvilket viser en loading animation i brugerfladen. Derefter kalder den SendMessage funktionen fra RequestPage.js med msg som parameter. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver sendt til OpenAI API'en.

Når API'en svarer, sætter funktionen loading til false, hvilket fjerner loading animationen fra brugerfladen. Derefter tjekker den, om API'en har svaret med en besked (response.data.BOT). Hvis den har, opretter den en besked med svaret fra API'en og tilføjer den til beskedhistorikken. Hvis den ikke har, opretter den en besked med en standardfejlmeddelelse og tilføjer den til beskedhistorikken.

#### renderBubble, renderInputToolbar og renderSend function
Vi laver nu en function der laver en bobble til vores chat. <br>

```javascript
//Custom Bubble
   const renderBubble =(props)=> {
        return (
          <Bubble
            {...props}
            wrapperStyle={{
              right: {
                backgroundColor: '#671ddf',
               
              },left:{
               
              }
             
            }}
            textStyle={{
                right:{
                    // fontSize:20,
                    padding:2
                },
                left: {
                  color: '#671ddf',
                  // fontSize:20,
                  padding:2
                }
              }}
          />
        )
      }
```

Vi laver nu en function der laver en toolbar til vores chat.

```javascript
const  renderInputToolbar =(props)=> {
        //Add the extra styles via containerStyle
       return <InputToolbar {...props} 
       containerStyle={{
       padding:3,
      
        backgroundColor:'#671ddf',
        color:'#fff',
        }} 
        
        textInputStyle={{ color: "#fff" }}
         />
     }
```

Vi laver nu en function der laver en send knap til vores chat.

```javascript
const  renderSend=(props)=> {
        return (
            <Send
                {...props}
            >
                <View style={{marginRight: 10, marginBottom: 5}}>
                <FontAwesome name="send" size={24} color="white" resizeMode={'center'} />
                   
                </View>
            </Send>
        );
    }
```

#### Return function
Til sidst skal vi nu lave en return function der indeholder vores chat. <br>

```javascript
//SafeAreaView er en container, der beskytter mod systembarer, keyboard og andre elementer, der kan dække ind - Specielt vigtig i de nye IOS versioner
  return (
    <SafeAreaView style={{flex: 1,backgroundColor:'#671ddf'}}>
        <View style={{ flex: 1,backgroundColor:'#fff' }}>

        <GiftedChat
        messages={messages}
        isTyping={loading}
        multiline ={true}
        onSend={messages => onSend(messages)}
        user={{
            _id: 1,
        
        }}
        renderBubble={renderBubble}
        renderInputToolbar={renderInputToolbar} 
        renderSend={renderSend}
        />
        
        
        </View>
    </SafeAreaView>

  )
```

ChatScreen.js skulle nu gerne være done.


I er nu done med jeres ChatBot. <B>Highfive! ᕦ(▀̿ ̿ -▀̿ ̿ )つ</B>

Her er en checkliste til at se om I har alle filerne:

# Fil Checkliste

- [ ] `StackNavigator.js`
- [ ] `App.js`
- [ ] `HomeScreen.js`
- [ ] `ChatScreen.js`
- [ ] `ChatFaceData.js`
- [ ] `Request.js`
