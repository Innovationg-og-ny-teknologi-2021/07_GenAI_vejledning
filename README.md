# 10_ChatBot__vejledning

I dag skal vi arbejde med at implementere en GPT-3 chatbot i en react-native applikation. Det er lidt mere komplekst end I er vant til, derfor har jeg prøvet en ny vejlednings metode med mere teori og forklaring. <br>

KOM ENDELIG MED FEEDBACK OM I KAN LIDE DENNE FORM FOR VEJLEDNING <3 <br>

## Opsætning af projekt

Start med at oprette et nyt react-native projekt med expo-cli. 

``` npx create-expo-app 10_ChatBot```

Vælg blank template og navigere til projektmappen med cd og kør projektet med ```npm start```

Når projektet er startet op, kan du åbne det i en simulator eller på din telefon. 
Sluk herefter serveren og installer de nødvendige komponenter med:

``` npm install @react-native-gesture-handler @react-native-gifted-chat @react-native-safe-area-context @react-navigation/native-stack @react-navigation/native @react-native-async-storage/async-storage @axios```

## Stack navigation

Vi skal bruge en stack navigation til at navigere mellem vores sider.
<br>
Opret en ny mappe kaldt komponents og i den mappe opret 3 filer 
1. StackNavigator.js
2. HomeScreen.js
3. ChatScreen.js

I StackNavigator.js skal vi importere de nødvendige komponenter og oprette en stack navigation.
Stack navigationen skal have 2 sider, en til vores chat og en til vores home screen.
<br>
Hvis du ikke kan huske hvordan man gør, så kig i [dokumentationen](https://reactnavigation.org/docs/stack-navigator/) eller i vores navigations øvelse.

## HomeScreen.js

Før vi kan arbejde med vores chatbot, skal vi have lavet en home screen hvor vi kan vælge at starte en chat.
<br>
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
import { useNavigation } from "@react-navigation/native";
import AsyncStorage from "@react-native-async-storage/async-storage";
```

### HomeScreen logik
I vores **HomeScreen function** skal vi oprette 2 useStates til at hente vores valgte chatbots.
Vi skal også hente vores navigation. 

```javascript
  //Håndtere ChatBotValg
  const [chatFaceData, setChatFaceData] = useState([]);
  const [selectedChatFace, setSelectedChatFace] = useState([]);
  //Får navigation til at virke
  const navgitaion = useNavigation();
```
Dette er en React Hook ved navn useState, der bruges til at oprette en tilstandsvariabel og en funktion til at opdatere denne tilstand. <br>

``chatFaceData`` er en tilstandsvariabel, som initialt er sat til et tomt array. <br>

``setChatFaceData`` er en funktion, der kan bruges til at opdatere værdien af chatFaceData.
<br>

Det samme gælder for ``selectedChatFace``.


Herefter laver vi en useEffect til at hente vores valgte chatbot.
```javascript
//Håndtere valg af chatbot
  useEffect(() => {
    setChatFaceData(ChatFaceData);
    checkFaceId();
  }, []);
```
useEffect er en hook i React, der giver mulighed for at udføre sideeffekter i funktionelle komponenter. <br>

Den tager to argumenter: en funktion, der indeholder den kode, der skal køres `` setChatFaceData ``, og en afhængighedsarray `` [] ``. <br>

Afhængighedsarrayen i useEffect bestemmer, hvornår effekten skal køres.

Hvis det er tomt (som det er i denne kode), vil useEffect kun køre én gang lige efter komponenten er monteret (lignende componentDidMount i klassebaserede komponenter).

Hvis der var værdier inde i denne array, ville useEffect køre, når en af disse værdier ændres.

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
Funktionsdefinition:

`` const checkFaceId = async () => {...}: `` Dette er en asynkron funktion defineret ved hjælp af arrow funktion syntax. async nøgleordet angiver, at funktionen vil indeholde asynkrone operationer, typisk dem, der venter på noget (f.eks. netværksopkald, databaseforespørgsler) ved hjælp af await. <br>

Hentning af data fra AsyncStorage:

`` const id = await AsyncStorage.getItem("chatFaceId");: ``  Her forsøger vi at hente en værdi fra AsyncStorage med nøglen "chatFaceId". AsyncStorage er en simpel, asynkron, persistent, nøgle-værdi lagringsmekanisme, som ofte bruges i React Native apps. <br> <br>
await nøgleordet betyder, at funktionen vil vente, indtil `` AsyncStorage.getItem("chatFaceId") `` er færdig med at hente den ønskede værdi, før den fortsætter. <br>

Efter at have hentet id, tjekker vi, om id har en sandhedsværdi (dvs. det er ikke null, undefined, false, 0, eller en tom streng). <Br>
`` id ? setSelectedChatFace(ChatFaceData[id]) : setSelectedChatFace(ChatFaceData[0]);: `` Dette er en ternær betingelsesoperatør. Den fungerer således: <br> <br>
Hvis id har en sandhedsværdi, vil `` setSelectedChatFace(ChatFaceData[id]) `` blive kaldt. Dette vil sætte selectedChatFace tilstanden til værdien fra ChatFaceData arrayet på den position, som id peger på.
Hvis id ikke har en sandhedsværdi (dvs. det er falskt), vil `` setSelectedChatFace(ChatFaceData[0]) `` blive kaldt i stedet. Dette vil sætte selectedChatFace tilstanden til den første værdi i ChatFaceData arrayet.


#### onChatFacePress function
Til sidst laver vi en funktion der håndtere at vi kan skifte imellem dem.
```javascript
//Håndtere valg af chatbot (Skifte mellem dem)
  const onChatFacePress = async (id) => {
    setSelectedChatFace(ChatFaceData[id - 1]);
    await AsyncStorage.setItem("chatFaceId", (id - 1).toString());
  };
```
Funktionen onChatFacePress tager en parameter id. Når den kaldes, opdaterer den tilstanden selectedChatFace baseret på id og gemmer derefter id - 1 som en streng i AsyncStorage under nøglen "chatFaceId". 

Denne funktion kan blive brugt, når en bruger klikker/tapper på et chat ansigt i en brugerflade, hvorefter det valgte ansigt bliver gemt for fremtidige referencer.


### Return function i HomeScreen
Vi skal lave 2 views til at wrappe resten af vores frontend. 
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

Koden fungere således: <br>
Vi bruger en Text komponent til at vise en titel, undertitel og en besked. <br>

Vi bruger også en Image komponent til at vise vores chatbot. <br>

Vi har nu lavet en titel, undertitel, billede og en besked. <br>

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

Koden fungere således: <br>
Vi bruger et View komponent til at vise en flatliste med vores chatbots. <br>

Vi bruger en FlatList komponent til at vise vores chatbots. <br>

Vi bruger en TouchableOpacity komponent til at vise vores chatbots. <br>


Til sidst skal vi lave en knap til chatten

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

Din HomeScreen er nu næsten done. <br>

### Hent vores ChatBots

Vi skal nu hente vores ChatBots fra en ny mappe. <br>

Lav en mappe i din komponents mappe og kald den **services**. <br>

I services mappen skal du oprette en fil kaldt **ChatFaceData.js**. <br>

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

Koden fungere således: <br>
Vi eksporterer et array af objekter ved navn chatFaceData. Hvert objekt indeholder data om en chatbot, herunder id, navn, billede, primær farve og sekundær farve. <br>

Vi bruger dette array til at vise en liste over chatbots i vores HomeScreen. <br>


Nu skal du importer ChatFaceData i din HomeScreen. <br>

```javascript
import ChatFaceData from "../services/ChatFaceData";
```

Du er nu done med HomeScreen. Check at det virker ved at kører koden med npm start <br>

## ChatScreen

### RequestPage.js

Start med at oprette en ny fil i din services mappe kaldt RequestPage.js. <br>

Gå nu til dette link: https://rapidapi.com/rphrp1985/api/open-ai21 og opret en bruger. <br>

Når du har oprettet en bruger, skal du trykke på "test endpoint" og kopiere din API nøgle. <br>

I RequestPage.js skal du indsætte følgende kode (dog med din egen API nøgle):
```javascript
// Sample import fra https://rapidapi.com/rphrp1985/api/open-ai21
// Hunsk at indsætte den sample de giver ind i en function som jeg har gjort. Husk også at filføje en return response
import axios from "axios";

export default async function SendMessage(message) {
  const options = {
    method: "POST",
    url: "https://open-ai21.p.rapidapi.com/conversationgpt35",
    headers: {
      "content-type": "application/json",
      "X-RapidAPI-Key": "cbc496968fmsh4191adf4200576bp1273efjsn40f86d869350",
      "X-RapidAPI-Host": "open-ai21.p.rapidapi.com",
    },
    data: {
      messages: [
        {
          role: "user",
          content: message,
        },
      ],
      web_access: false,
      stream: false,
    },
  };

  try {
    const response = await axios.request(options);
    console.log(response.data);
    return response
  } catch (error) {
    console.error(error);
  }
}
```

Koden fungere således: <br>
Vi importerer axios fra axios. Axios er et HTTP-klientbibliotek, der bruges til at foretage HTTP-anmodninger fra node.js eller XMLHttpRequests fra browseren. <br>

Vi eksporterer en asynkron funktion ved navn SendMessage, der tager en parameter message. Når den kaldes, opretter den en konstant options, der indeholder de nødvendige oplysninger til at foretage en POST-anmodning til OpenAI API'en. <br>

Derefter forsøger den at foretage en POST-anmodning til OpenAI API'en ved hjælp af axios.request. Hvis anmodningen lykkes, returnerer den svaret fra API'en. Hvis anmodningen mislykkes, udskriver den en fejl i konsollen. <br>


### ChatScreen.js
Vi skal nu oprette en ny fil i vores komponents mappe kaldt ChatScreen.js. <br>

I ChatScreen.js skal du importer følgende:
```javascript
import { View, Text, Image } from 'react-native'
import React from 'react'
//GiftedChat er det vi bruger til at lave chatten
import { Bubble, GiftedChat, InputToolbar, Send } from 'react-native-gifted-chat'

import { useState } from 'react';
import { useEffect } from 'react';
import { useCallback } from 'react';
import { FontAwesome } from '@expo/vector-icons';
import ChatFaceData from './services/ChatFaceData';
import AsyncStorage from '@react-native-async-storage/async-storage';
import SendMessage from './services/RequestPage';

//Håndtere hvis der er fejl i AsyncStorage
CHAT_BOT_FACE='https://res.cloudinary.com/dknvsbuyy/image/upload/v1685678135/chat_1_c7eda483e3.png'
```

Vi skal nu lave 3 useStates til at håndtere vores chat. <br>

```javascript
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
Funktionen checkFaceId tjekker, om der er gemt en chatbot-ID i AsyncStorage under nøglen 'chatFaceId'.

Hvis der er en ID gemt, bliver den specifikke chatbots billede og primære farve sat fra ChatFaceData arrayet baseret på denne ID.

Hvis der ikke er en gemt ID, bliver den første chatbots billede i ChatFaceData arrayet anvendt som standard.

Derefter opdaterer funktionen tilstanden chatFaceColor med den primære farve af den valgte chatbot.

Slutteligt initialiserer den beskedhistorikken (setMessages) med en enkelt besked fra den valgte chatbot, der introducerer sig selv med sit navn og spørger, hvordan den kan hjælpe. Beskeden indeholder metadata som ID, oprettelsestidspunkt og avatar (som er chatbotens billede).

Kort sagt, checkFaceId funktionen tjekker, hvilken chatbot der tidligere er blevet valgt (hvis nogen), sætter chatbotens visuelle repræsentation, og initialiserer en introduktionsbesked fra den valgte chatbot.


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

Funktionen onSend tager en parameter messages. Når den kaldes, opdaterer den beskedhistorikken (setMessages) ved at tilføje de nye beskeder til den eksisterende beskedhistorik. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver tilføjet til beskedhistorikken.


#### getBardResp function
Vi skal nu lave en function der håndtere når vi modtager en besked fra vores bot <br>

```javascript
//Håndtere API Kald og BOT Svar
    const getBardResp = (msg) => {
        setLoading(true)
        //SendMessage er vores funktion fra RequestPage.js
        SendMessage(msg)
        .then(response => {
            // Extracting the AI's reply from `response.data.BOT`
            if (response.data && response.data.BOT) {
                setLoading(false)
                
                const chatAIResp = {
                    _id: Math.random() * (9999999 - 1),
                    text: response.data.BOT,
                    createdAt: new Date(),
                    user: {
                      _id: 2,
                      name: 'React Native',
                      avatar: CHAT_BOT_FACE,
                    }
                }
    
                setMessages(previousMessages => GiftedChat.append(previousMessages, chatAIResp))  
            } else {
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

Funktionen getBardResp tager en parameter msg. Når den kaldes, sætter den tilstanden loading til true, hvilket viser en loading animation i brugerfladen. Derefter kalder den SendMessage funktionen fra RequestPage.js med msg som parameter. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver sendt til OpenAI API'en. <br>

Når API'en svarer, sætter funktionen loading til false, hvilket fjerner loading animationen fra brugerfladen. Derefter tjekker den, om API'en har svaret med en besked (response.data.BOT). Hvis den har, opretter den en besked med svaret fra API'en og tilføjer den til beskedhistorikken. Hvis den ikke har, opretter den en besked med en standardfejlmeddelelse og tilføjer den til beskedhistorikken. <br>

#### renderBubble, renderInputToolbar og renderSend function
Vi laver nu en function der laver en bobble til vores chat. <br>

```javascript
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

Funktionen renderBubble tager en parameter props. Når den kaldes, returnerer den en Bubble komponent fra react-native-gifted-chat. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver vist i en bobble. <br>


Vi laver nu en function der laver en toolbar til vores chat. <br>

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

Funktionen renderInputToolbar tager en parameter props. Når den kaldes, returnerer den en InputToolbar komponent fra react-native-gifted-chat. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver vist i en toolbar. <br>


Vi laver nu en function der laver en send knap til vores chat. <br>

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

Funktionen renderSend tager en parameter props. Når den kaldes, returnerer den en Send komponent fra react-native-gifted-chat. Denne funktion kan blive brugt, når en bruger sender en besked i en brugerflade, hvorefter beskeden bliver vist i en send knap. <br>


#### Return function
Til sidst skal vi nu lave en return function der indeholder vores chat. <br>

```javascript
 return (
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
  )
```

ChatScreen.js skulle nu gerne være done.

### App.js

Vi skal nu tilbage til vores App.js og importere vores StackNavigator. <br>

```javascript
return <HomeNavigation />;
```

I skal også slette jeres styles i App.js og huske at importer de nødvendige komponenter <br>

I er nu done med jeres ChatBot. Highfive! <br>