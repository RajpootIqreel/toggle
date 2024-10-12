# toggle
import React, { useState, useEffect } from 'react';
import {
  StyleSheet,
  Text,
  View,
  Button,
  FlatList,
  Alert,
  Switch,
  AsyncStorage,
} from 'react-native';

const quizData = [
  {
    question: "What's the capital of France?",
    options: ['Berlin', 'Madrid', 'Paris', 'Lisbon'],
    answer: 'Paris',
  },
  {
    question: "What's the capital of Germany?",
    options: ['Berlin', 'Vienna', 'Brussels', 'Amsterdam'],
    answer: 'Berlin',
  },
  {
    question: "What's the capital of Spain?",
    options: ['Madrid', 'Barcelona', 'Sevilla', 'Valencia'],
    answer: 'Madrid',
  },
  {
    question: "What's the largest ocean on Earth?",
    options: ['Atlantic', 'Indian', 'Arctic', 'Pacific'],
    answer: 'Pacific',
  },
  {
    question: "Which planet is known as the Red Planet?",
    options: ['Earth', 'Mars', 'Jupiter', 'Saturn'],
    answer: 'Mars',
  },
];

export default function App() {
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);
  const [score, setScore] = useState(0);
  const [isQuizFinished, setIsQuizFinished] = useState(false);
  const [timer, setTimer] = useState(30);
  const [darkMode, setDarkMode] = useState(false);
  const [intervalId, setIntervalId] = useState(null);

  useEffect(() => {
    loadTheme();
  }, []);

  useEffect(() => {
    if (timer > 0 && !isQuizFinished) {
      const id = setInterval(() => setTimer((prev) => prev - 1), 1000);
      setIntervalId(id);
    } else if (timer === 0) {
      handleNextQuestion();
    }
    return () => clearInterval(intervalId);
  }, [timer, isQuizFinished]);

  const loadTheme = async () => {
    const savedTheme = await AsyncStorage.getItem('darkMode');
    if (savedTheme !== null) {
      setDarkMode(JSON.parse(savedTheme));
    }
  };

  const toggleDarkMode = async () => {
    const newMode = !darkMode;
    setDarkMode(newMode);
    await AsyncStorage.setItem('darkMode', JSON.stringify(newMode));
  };

  const handleAnswer = (selectedOption) => {
    if (selectedOption === quizData[currentQuestionIndex].answer) {
      setScore(score + 1);
    }
    handleNextQuestion();
  };

  const handleNextQuestion = () => {
    if (currentQuestionIndex < quizData.length - 1) {
      setCurrentQuestionIndex(currentQuestionIndex + 1);
      setTimer(30); // Reset timer for next question
    } else {
      setIsQuizFinished(true);
      clearInterval(intervalId);
    }
  };

  const resetQuiz = () => {
    setCurrentQuestionIndex(0);
    setScore(0);
    setIsQuizFinished(false);
    setTimer(30);
  };

  const backgroundStyle = {
    backgroundColor: darkMode ? '#333' : '#FFF',
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  };

  const textStyle = {
    color: darkMode ? '#FFF' : '#000',
  };

  return (
    <View style={backgroundStyle}>
      <Switch
        value={darkMode}
        onValueChange={toggleDarkMode}
        style={{ marginBottom: 20 }}
      />
      <Text style={[textStyle, styles.title]}>Quiz App</Text>

      {!isQuizFinished ? (
        <>
          <Text style={[textStyle, styles.question]}>
            {quizData[currentQuestionIndex].question}
          </Text>
          <Text style={[textStyle, styles.timer]}>Time Remaining: {timer}s</Text>
          <FlatList
            data={quizData[currentQuestionIndex].options}
            keyExtractor={(item) => item}
            renderItem={({ item }) => (
              <Button title={item} color={darkMode ? '#FFF' : '#000'} onPress={() => handleAnswer(item)} />
            )}
          />
        </>
      ) : (
        <View style={styles.result}>
          <Text style={[textStyle, styles.resultText]}>
            Quiz Finished! Your Score: {score}/{quizData.length}
          </Text>
          <Button title="Restart Quiz" onPress={resetQuiz} />
          <Text style={[textStyle, styles.correctAnswersTitle]}>Correct Answers:</Text>
          {quizData.map((question, index) => (
            <Text key={index} style={textStyle}>
              {index + 1}. {question.question} - Correct Answer: {question.answer}
            </Text>
          ))}
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  question: {
    fontSize: 20,
    marginBottom: 20,
  },
  timer: {
    fontSize: 16,
    marginBottom: 20,
    color: 'red',
  },
  result: {
    alignItems: 'center',
  },
  resultText: {
    fontSize: 24,
    marginBottom: 20,
  },
  correctAnswersTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 20,
  },
});
