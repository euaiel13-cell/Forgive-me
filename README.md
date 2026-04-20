import React, { useEffect, useMemo, useState } from "react";
import { AnimatePresence, motion } from "framer-motion";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

const questions = [
  {
    id: 1,
    text: 'Do you forgive me?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 2,
    text: 'Do you love me?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 3,
    text: 'Do you care for me?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 4,
    text: 'Do you miss me?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 5,
    text: 'Can you imagine a life without me?',
    correctAnswer: 'no',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 6,
    text: 'Do you truly forgive me?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOO TRY AGAIN',
  },
  {
    id: 7,
    text: 'Are you open to seeing me again?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
  {
    id: 8,
    text: 'Whenever you get back, allow me the pleasure of taking you out somewhere special. What do you say?',
    correctAnswer: 'yes',
    wrongMessage: 'WRONG ANSWER BOOOOO TRY AGAIN',
  },
];

const apologyMessage = `I am sorry for everything I have done. I know I messed up I made one mistake and it was stupid. Please do not let that one mistake define me and how well it was going. I care for you abundantly and I may have done a poor job of showing it but I never stopped caring or loving you I was in my head. But what is even stupider is you thinking you can get rid of me again lol. You know where home is #no Uber or lyft.`;

// To actually receive emails, connect this to your own backend/webhook.
// Example options: a small server endpoint, Formspree, Zapier webhook, Make, or EmailJS.
const EMAIL_WEBHOOK_URL = "";

function FloatingItem({ item, emoji }) {
  return (
    <motion.div
      className="pointer-events-none fixed select-none text-3xl md:text-5xl"
      initial={{
        opacity: 0,
        scale: 0.4,
        x: item.x,
        y: item.y,
        rotate: item.rotate,
      }}
      animate={{
        opacity: [0, 1, 1, 0],
        scale: [0.4, 1, 1.1, 0.8],
        y: [item.y, item.y - item.floatBy],
        x: [item.x, item.x + item.drift],
        rotate: item.rotate + 40,
      }}
      transition={{
        duration: item.duration,
        ease: 'easeOut',
      }}
      style={{ left: 0, top: 0 }}
    >
      {emoji}
    </motion.div>
  );
}

export default function ForgiveMeWebsite() {
  const [currentIndex, setCurrentIndex] = useState(0);
  const [showWrongMessage, setShowWrongMessage] = useState(false);
  const [recordedAnswers, setRecordedAnswers] = useState({});
  const [attemptLog, setAttemptLog] = useState([]);
  const [showApology, setShowApology] = useState(false);
  const [finished, setFinished] = useState(false);
  const [floatingItems, setFloatingItems] = useState([]);
  const [emailStatus, setEmailStatus] = useState("idle");

  const currentQuestion = questions[currentIndex];

  const celebrationEmojis = useMemo(() => ['🌸', '🌷', '🌹', '💐', '🌺', '🌼', '💖', '💕', '💘', '❤️'], []);

  const burstCelebration = () => {
    const count = 56;
    const newItems = Array.from({ length: count }, (_, i) => ({
      id: `${Date.now()}-${i}`,
      x: Math.random() * window.innerWidth,
      y: Math.random() * window.innerHeight,
      rotate: Math.random() * 360,
      drift: (Math.random() - 0.5) * 180,
      floatBy: 60 + Math.random() * 220,
      duration: 2.2 + Math.random() * 1.8,
      emoji: celebrationEmojis[Math.floor(Math.random() * celebrationEmojis.length)],
    }));

    setFloatingItems((prev) => [...prev, ...newItems]);

    setTimeout(() => {
      setFloatingItems([]);
    }, 5000);
  };

  useEffect(() => {
    if (finished) {
      burstCelebration();
    }
  }, [finished]);

  const sendResponses = async (finalAttemptLog, finalRecordedAnswers) => {
    if (!EMAIL_WEBHOOK_URL) {
      setEmailStatus("not_configured");
      return;
    }

    try {
      setEmailStatus("sending");

      const payload = {
        submittedAt: new Date().toISOString(),
        finalAnswers: finalRecordedAnswers,
        attempts: finalAttemptLog,
      };

      const response = await fetch(EMAIL_WEBHOOK_URL, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(payload),
      });

      if (!response.ok) {
        throw new Error("Failed to send responses");
      }

      setEmailStatus("sent");
    } catch (error) {
      console.error("Email delivery failed:", error);
      setEmailStatus("failed");
    }
  };

  const handleAnswer = async (answer) => {
    if (!currentQuestion) return;

    const isCorrect = answer === currentQuestion.correctAnswer;

    const updatedAttemptLog = [
      ...attemptLog,
      {
        id: `${currentQuestion.id}-${attemptLog.length}-${Date.now()}`,
        question: currentQuestion.text,
        answer,
        result: isCorrect ? 'Accepted' : 'Wrong answer',
      },
    ];

    setAttemptLog(updatedAttemptLog);

    if (!isCorrect) {
      setShowWrongMessage(true);
      setTimeout(() => setShowWrongMessage(false), 1200);
      return;
    }

    const updatedRecordedAnswers = {
      ...recordedAnswers,
      [currentQuestion.id]: answer,
    };

    setRecordedAnswers(updatedRecordedAnswers);

    if (currentQuestion.id === 7) {
      setShowApology(true);
      setCurrentIndex(7);
      return;
    }

    if (currentQuestion.id === 8) {
      setFinished(true);
      await sendResponses(updatedAttemptLog, updatedRecordedAnswers);
      return;
    }

    setCurrentIndex((prev) => prev + 1);
  };

  const restart = () => {
    setCurrentIndex(0);
    setShowWrongMessage(false);
    setRecordedAnswers({});
    setAttemptLog([]);
    setShowApology(false);
    setFinished(false);
    setFloatingItems([]);
    setEmailStatus("idle");
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-pink-100 via-rose-50 to-red-100 px-4 py-10 relative overflow-hidden">
      <AnimatePresence>
        {floatingItems.map((item) => (
          <FloatingItem key={item.id} item={item} emoji={item.emoji} />
        ))}
      </AnimatePresence>

      <div className="mx-auto flex min-h-[85vh] max-w-3xl items-center justify-center">
        <Card className="w-full rounded-3xl border-0 bg-white/85 shadow-2xl backdrop-blur">
          <CardContent className="p-6 md:p-10">
            <AnimatePresence mode="wait">
              {!finished ? (
                <motion.div
                  key={`question-${currentQuestion?.id}`}
                  initial={{ opacity: 0, y: 18 }}
                  animate={{ opacity: 1, y: 0 }}
                  exit={{ opacity: 0, y: -18 }}
                  transition={{ duration: 0.3 }}
                  className="text-center"
                >
                  <div className="mb-3 text-sm font-medium uppercase tracking-[0.25em] text-rose-400">
                    One question at a time
                  </div>

                  <h1 className="mb-6 text-2xl font-bold leading-tight text-slate-800 md:text-4xl">
                    {currentQuestion?.text}
                  </h1>

                  {showWrongMessage && (
                    <motion.div
                      initial={{ opacity: 0, scale: 0.9 }}
                      animate={{ opacity: 1, scale: 1 }}
                      className="mb-6 rounded-2xl border border-red-200 bg-red-50 px-4 py-3 text-sm font-semibold text-red-600 md:text-base"
                    >
                      {currentQuestion?.wrongMessage}
                    </motion.div>
                  )}

                  {showApology && currentQuestion?.id === 8 && (
                    <motion.div
                      initial={{ opacity: 0, y: 14 }}
                      animate={{ opacity: 1, y: 0 }}
                      className="mx-auto mb-8 max-w-2xl rounded-3xl bg-rose-50 p-5 text-left text-slate-700 shadow-sm"
                    >
                      <p className="text-base leading-7 md:text-lg">
                        {apologyMessage}
                      </p>
                    </motion.div>
                  )}

                  <div className="flex flex-col justify-center gap-4 sm:flex-row">
                    <Button
                      onClick={() => handleAnswer('yes')}
                      className="rounded-2xl px-8 py-6 text-base md:text-lg"
                    >
                      Yes
                    </Button>
                    <Button
                      onClick={() => handleAnswer('no')}
                      variant="outline"
                      className="rounded-2xl px-8 py-6 text-base md:text-lg"
                    >
                      No
                    </Button>
                  </div>
                </motion.div>
              ) : (
                <motion.div
                  key="finished"
                  initial={{ opacity: 0, y: 18 }}
                  animate={{ opacity: 1, y: 0 }}
                  className="text-center"
                >
                  <div className="mb-4 text-6xl">💖</div>
                  <h1 className="mb-4 text-3xl font-bold text-slate-800 md:text-5xl">
                    Response has been recorded, see you at our date ma heart!
                  </h1>
                  <p className="mx-auto mb-4 max-w-xl text-base leading-7 text-slate-600 md:text-lg">
                    Flowers and hearts have officially taken over the screen.
                  </p>

                  {emailStatus === "sending" && (
                    <p className="mx-auto mb-4 text-sm font-medium text-slate-500">
                      Sending responses...
                    </p>
                  )}

                  {emailStatus === "sent" && (
                    <p className="mx-auto mb-4 text-sm font-semibold text-green-600">
                      Responses sent successfully.
                    </p>
                  )}

                  {emailStatus === "failed" && (
                    <p className="mx-auto mb-4 text-sm font-semibold text-red-600">
                      Could not send responses. Check your webhook setup.
                    </p>
                  )}

                  {emailStatus === "not_configured" && (
                    <p className="mx-auto mb-4 text-sm font-semibold text-amber-600">
                      Email sending is not connected yet.
                    </p>
                  )}
                  <div className="flex flex-col justify-center gap-4 sm:flex-row">
                    <Button onClick={burstCelebration} className="rounded-2xl px-8 py-6 text-base md:text-lg">
                      More flowers + hearts
                    </Button>
                    <Button
                      onClick={restart}
                      variant="outline"
                      className="rounded-2xl px-8 py-6 text-base md:text-lg"
                    >
                      Start again
                    </Button>
                  </div>
                </motion.div>
              )}
            </AnimatePresence>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
