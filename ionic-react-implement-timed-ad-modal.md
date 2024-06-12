# Implementing a Timed Ad Modal in Ionic React

In modern web and mobile applications, advertisements are a common way to generate revenue. However, ensuring that users engage with ads for a sufficient amount of time can be a challenge. One effective solution is to use a timed modal that controls how long an ad is displayed before the user can close it. In this article, we'll walk through how to implement a timed ad modal in an Ionic React application.

## Overview

We'll create an `AdModal` component in an Ionic React app that includes a countdown timer. This modal will display an advertisement and prevent the user from closing it until 15 seconds have passed, ensuring that the ad has a chance to capture the user's attention.

## Setting Up the Ionic React Project

First, ensure you have Ionic installed. If not, you can set it up with the following commands:

```bash
npx create-react-app my-app --template @ionic/react
cd my-app
```

## Creating the AdModal Component

The AdModal component will use Ionic components like IonModal, IonButton, and IonContent. We'll add state management to handle the countdown and control the modal's behavior.

## omponent Code

Create a file named AdModal.js in the src/components folder with the following code:

```jsx
import React, { useState, useEffect } from "react";
import {
  IonModal,
  IonButton,
  IonContent,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonLabel,
} from "@ionic/react";

const AdModal = ({ isOpen, onClose }) => {
  const [enableClose, setEnableClose] = useState(false);
  const [counter, setCounter] = useState(15); // Initialize counter at 15 seconds

  useEffect(() => {
    let interval = null;

    if (isOpen && !enableClose) {
      interval = setInterval(() => {
        setCounter((prevCounter) => {
          if (prevCounter <= 1) {
            clearInterval(interval); // Clear interval when counter reaches 0
            setEnableClose(true); // Enable the close button
            return 0;
          }
          return prevCounter - 1;
        });
      }, 1000);
    }

    return () => {
      clearInterval(interval); // Clear interval on component unmount or when modal closes
      setCounter(15); // Reset counter
      setEnableClose(false); // Reset enableClose state
    };
  }, [isOpen, enableClose]);

  return (
    <IonModal isOpen={isOpen} cssClass="my-custom-class">
      <IonHeader>
        <IonToolbar>
          <IonTitle>Ad Space</IonTitle>
          <IonButton slot="end" onClick={onClose} disabled={!enableClose}>
            Close
          </IonButton>
        </IonToolbar>
      </IonHeader>
      <IonContent className="ion-padding">
        <h2>This is your Ad Space</h2>
        <p>
          Insert your ad content here, whether it's an image, video, or text.
        </p>
        {!enableClose && (
          <IonLabel>Ad will close in {counter} seconds...</IonLabel>
        )}
      </IonContent>
    </IonModal>
  );
};

export default AdModal;
```

## Integration into Your App

Integrate the AdModal into your main app component to manage its visibility:

```jsx
import React, { useState } from "react";
import { IonApp, IonButton } from "@ionic/react";
import AdModal from "./components/AdModal"; // Adjust the import path based on your file structure

const App = () => {
  const [showModal, setShowModal] = useState(false);

  const handleOpenModal = () => {
    setShowModal(true);
  };

  const handleCloseModal = () => {
    setShowModal(false);
  };

  return (
    <IonApp>
      <IonButton onClick={handleOpenModal}>Show Ad</IonButton>
      <AdModal isOpen={showModal} onClose={handleCloseModal} />
    </IonApp>
  );
};

export default App;
```

## Conclusion

This approach ensures that ads are given a fair amount of screen time, potentially increasing ad effectiveness. The AdModal can be customized further to fit different types of ads and integrate with ad serving platforms. Experiment with different styles, placements, and content to find what works best for your audience and business model.
